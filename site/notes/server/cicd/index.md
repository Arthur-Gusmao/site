# CI/CD and pull requests

[Server guide](/notes/server/)

## GitHub Actions

`.github/workflows/container.yml` defines `CI / image`. It runs on PRs, master
pushes, and manual dispatch, using a GitHub-hosted Ubuntu runner. The job has a
15-minute timeout; newer runs cancel older runs in the same PR or branch group.
Checkout includes the pinned werc submodule and does not persist credentials.
Podman must report rootless execution.

CI checks shell syntax, builds the Containerfile, validates the werc and Plan 9
executables and lighttpd configuration, and checks the core pages and assets over
HTTP. The private config must be absent from the image and return HTTP 403 when
requested. The revision marker must exactly match `GITHUB_SHA`.

PRs test without publishing. A passing master run pushes
`ghcr.io/arthur-gusmao/site:FULL_COMMIT_SHA` using the job's GitHub token.
The image job requests package write permission; fork token permissions remain
subject to GitHub policy. There is no notebook-hosted runner or deployment secret.
On PR events the tested SHA is normally GitHub's merge commit; see
[GitHub's event reference](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows).

Requiring this check before merging is a separate branch-protection setting.
The workflow alone does not enforce merge policy.

## User cron

`/etc/crontabs/werc` runs this once per minute as the service user:

    SHELL=/bin/sh
    * * * * * /usr/local/sbin/werc-deploy 2>&1 | /usr/bin/logger -t werc-deploy

The script uses one nonblocking flock. `--close` prevents containers from
inheriting the lock. An overlapping check exits harmlessly.
The automatic path respects `/etc/werc/paused` and reads master over public HTTPS
with a bounded Git query. A stopped production container is started first.
If its image revision already matches master, the check returns.

## The complete update

1. Pull the SHA-tagged image and validate its OCI revision label.
2. Run `werc-site.check` on port 18086 and wait for its revision marker.
3. Require HTTP 200 for the homepage, about, projects, notes, CSS, and font; require 403 for `/_werc/config`.
4. Remove the temporary candidate. Recheck master before an automatic replacement.
5. Remove the old stopped backup, stop production, and rename it to `werc-site.previous`.
6. Start the new `werc-site` on port 18085 and verify its HTTP revision.
7. On failure during replacement, restore and start the previous container.

The restart window begins when production stops. Caddy stays unchanged.
The stopped backup remains for explicit rollback. A rejected candidate leaves
production and its prior backup untouched.

Candidate and production readiness each allow ten attempts. The deployment
verifies loopback HTTP, not the public tunnel. A network outage should not undo
a locally healthy website release. Check the public domain separately.

An unavailable image or failed update is logged and retried on later checks.
Updates can skip intermediate commits when master changes quickly. Downloading
and validating adds time beyond the minute interval.
