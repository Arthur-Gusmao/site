# CI/CD and pull requests

[Server guide](/notes/server/)

## Workflow events

`.github/workflows/container.yml` defines `CI` with one job, `image`.
It runs for pushes to `master`, pull requests, and manual workflow dispatch.
A run has a 15-minute timeout. Concurrency is grouped by PR number or branch;
a newer run cancels the older run in that group.

The runner is GitHub-hosted `ubuntu-latest`. Checkout includes the pinned werc
submodule and disables persisted Git credentials. The job installs Podman if
needed and asserts that it is running rootless.

The workflow has read access to repository contents. The image job also requests
package write permission for publishing. Fork PR token permissions are controlled
by GitHub. The publish step runs only outside a PR on `refs/heads/master`.
There is no self-hosted runner on the notebook.

On a PR, `GITHUB_SHA` identifies GitHub's test merge commit, rather than always
being the contributor's branch-tip SHA. See [GitHub's workflow-event reference](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows).

## Build and validation

The image name is `ghcr.io/arthur-gusmao/site:GITHUB_SHA`. The full 40-character
revision is supplied as a build argument, written to `/pub/meta/revision.txt`,
and added to the OCI revision label. The Containerfile rejects malformed SHAs.

CI checks shell syntax, builds the image, verifies werc and Plan 9 executables,
checks source content, confirms the private site config is absent, and validates
lighttpd's configuration. Its HTTP test checks the homepage heading, page routes,
CSS, font, revision marker, and HTTP `403` for `/_werc/config`.

PRs build and test without publishing or deploying. A passing run on `master`
logs into GHCR with the job's `GITHUB_TOKEN` and pushes the SHA-tagged image.
No custom deployment secret or notebook SSH credential is required.

Branch protection is a separate repository setting. Configure a required
`CI / image` check if merges must be blocked until testing passes; defining
this workflow alone does not enforce that policy.

## Scheduled deployment

Root's cron entry is:

    * * * * * /usr/local/sbin/werc-oci-deploy 2>&1 | /usr/bin/logger -t werc-deploy

The root-owned deploy script changes to `/` so the service user can traverse
its working directory. It takes an exclusive nonblocking lock. `flock --close`
keeps the lock in the parent process rather than passing it to long-lived
container processes. A concurrent run exits without starting another deployment.

If `/etc/werc-oci/paused` exists, the check returns immediately, including before
its stopped-container recovery step. Otherwise it recovers a stopped active
container, reads `master` with a bounded `git ls-remote`, and compares the SHA
with the active container's revision label. It pulls only when the revision differs.

An unavailable image leaves production unchanged and is retried later. The script
checks `master` again after staging and refuses to promote an overtaken revision.
Checking happens once per minute; download, testing, and public verification
add time. Skipped intermediate commits are normal when changes arrive quickly.

## Staging

The staging script runs as `werc` through `su-exec`. It validates the slot,
SHA, image label, state marker, and Caddy upstream, and refuses to replace the
active slot. It also coordinates with the promotion lock.

If an inactive container exists, it is stopped and renamed to `SLOT.previous`.
The candidate takes its normal name and host port. The script retries homepage
readiness up to 15 times, checks all core routes and assets, verifies the revision,
and confirms the container is still running.

On failure it removes the candidate, restores the previous name, and restarts
that container if it had been running. On success it removes the temporary
previous inactive container. The active production container stays untouched.

## Promotion

The root-owned promotion script takes its own lock and refuses a concurrent
stage. It checks that the recorded active slot matches Caddy, validates the
candidate's HTTP responses, and saves one `Caddyfile.previous`.

It prepares and validates a candidate configuration, atomically replaces the
slot marker and Caddyfile, reloads Caddy, and verifies the revision through both
local Caddy and the public domain. Public revision verification allows three
attempts. A final public homepage request must also succeed.

A failed promotion restores the prior Caddyfile and slot marker and reloads the
old configuration. Reload failures are reported as urgent recovery errors.
The previous production container remains the rollback target. See
[Caddy's command-line documentation](https://caddyserver.com/docs/command-line).
