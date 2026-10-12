# Audit the server

[Server guide](/notes/server/)

The automatic path is one program running as one unprivileged user:

    user cron -> werc-deploy -> rootless Podman

Caddy always points to the same port. No program changes its upstream during
an update. There is no root promotion step, active-slot marker, multiple
transaction locks, sourced shell library, eval command, or deployment webhook.

## Read the code in this order

1. [werc-podman](https://github.com/Arthur-Gusmao/site/blob/master/ops/werc-podman): who runs Podman and where it stores data.
2. [werc-deploy](https://github.com/Arthur-Gusmao/site/blob/master/ops/werc-deploy): the complete update, test, and rollback flow.
3. [werc-site-openrc](https://github.com/Arthur-Gusmao/site/blob/master/ops/werc-site-openrc): start, stop, and HTTP verification.
4. [werc-install](https://github.com/Arthur-Gusmao/site/blob/master/ops/werc-install): the privileged one-time setup.

Then read the small Caddyfile, tunnel service, Containerfile, and CI workflow.
All files are listed in the [reference](/notes/server/reference/).

## Where to change behavior

The deployment script begins with ordinary variables for container names,
production/test ports, hostname, and image repository. Its next sections handle
identity, lock, interrupted recovery, revision selection, cleanup, candidate
testing, and production replacement in execution order.

There is one readiness function and one exit cleanup function. The cleanup flag
`replacing` becomes true just before production is stopped. `had_old` records
whether there was an existing production container to preserve. Container names
provide the remaining state; there is no parallel marker file to synchronize.

Change the port in Caddy and the service verifier too if changing the production
port. Changing the hostname also requires reviewing the Containerfile, lighttpd
configuration, CI HTTP tests, site templates, and Cloudflare route. This is a
single-site program, not a generic configurable platform.

## Invariants

- Deployment runs as werc, never as root.
- The fixed production name and port agree with Caddy and OpenRC.
- A full hexadecimal SHA matches the image label and HTTP revision.
- A failed candidate leaves production and its previous backup untouched.
- Replacement stops and renames production before creating its successor.
- Failed replacement restores the stopped previous container when it exists.
- The old backup is stopped and is not removed until the candidate has passed.
- The lock stays in the parent process rather than in a container supervisor.
- An automatic update rechecks master before replacing production.
- Manual rollback uses the previous container's exact local image ID.

## Check your installed version

From a reviewed checkout:

    git status --short
    git diff --check
    shellcheck -S warning ops/werc-* ops/cloudflared-openrc
    doas cmp ops/werc-deploy /usr/local/sbin/werc-deploy
    doas cmp ops/werc-podman /usr/local/sbin/werc-podman
    doas cmp ops/werc-site-openrc /etc/init.d/werc-site
    doas rc-service werc-site verify
    curl -fsS https://arthurgusmao.com.br/pub/meta/revision.txt

ShellCheck is optional for development, not a runtime dependency.
The public revision identifies the website image, not the installed host scripts.
Compare files when auditing host code. Read the actual source before changing it;
line count alone does not establish correctness or understandability.
