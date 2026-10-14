# Installation

[Server guide](/notes/server/) contains the fresh-install commands.
The installer requires Alpine Linux, OpenRC and root via `doas`.
It installs Podman, a dedicated user, services and the update cron job.
It downloads a pinned, checksum-verified cloudflared binary if missing.

## Migrate from the old Caddy setup

Wait for any deployment to finish. Pause updates, install the new host
scripts, and deploy the desired full commit SHA:

    doas touch /etc/werc/paused
    doas sh ops/werc-install
    doas werc update FULL_COMMIT_SHA
    doas werc resume

The installer stops and disables Caddy only when `/etc/caddy/Caddyfile`
matches this repository's legacy `ops/Caddyfile`. The new container uses
`8080` instead of `18085`; the tunnel route remains unchanged.
Expect an outage between stopping Caddy and completing the update.
Resolve any other service occupying port `8080` before installing.

Replaced host scripts are backed up in `/etc/werc/previous-install`.
The installer owns the dedicated `werc` crontab.

If migration fails, the previous container still uses port `18085`.
Restore access with `doas rc-service caddy start`, fix the failure, stop Caddy
again, then retry `doas werc update FULL_COMMIT_SHA` before resuming updates.
