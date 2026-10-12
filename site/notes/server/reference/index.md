# Files and commands

[Server guide](/notes/server/)

## The complete host code

- `ops/werc-deploy`: candidate testing, replacement, error recovery, and explicit rollback.
- `ops/werc-podman`: nine-line wrapper enforcing service identity and fixed storage paths.
- `ops/werc-site-openrc`: starts, stops, and verifies production before Caddy.
- `ops/werc-install`: installs packages, the user, mappings, scripts, configuration, and cron.
- `ops/cloudflared-openrc`: supervises the tunnel client.
- `ops/Caddyfile`: fixed loopback proxy; deployment never rewrites it.
- `ops/README.md`: repository quickstart.

Review the [actual source](https://github.com/Arthur-Gusmao/site/tree/master/ops).
Installed executable scripts are root-owned mode 0755 under `/usr/local/sbin`.
Service definitions are root-owned under `/etc/init.d`.

## Paths

- `/var/lib/werc`: private service home, owned by werc, mode 0700.
- `/var/lib/werc/.local/share/werc-oci/storage`: persistent Podman store.
- `/run/werc-oci`: service-owned private runtime, created by OpenRC.
- `/run/werc-oci/storage`: Podman runroot.
- `/run/werc-oci/libpod`: Podman temporary runtime.
- `/run/werc-oci/deploy.lock`: the single deployment lock.
- `/etc/werc/paused`: pauses automatic checks; explicit manual commands still run.
- `/etc/werc/previous-install`: one prior installed script/configuration set.
- `/etc/caddy/Caddyfile`: fixed proxy to 127.0.0.1:18085.
- `/etc/crontabs/werc`: the dedicated service user's cron job, installed as root with mode 0600.
- `/etc/cloudflared/token`: private root-owned credential, mode 0600, in a mode-0700 directory.
- `/etc/subuid` and `/etc/subgid`: subordinate user/group mappings.

The existing storage paths retain the `werc-oci` name to avoid changing Podman's
recorded database paths. There is no active-slot file, promotion lock, staging
lock, or separate deployment-state database.

## Containers

`werc-site` runs production on 18085. `werc-site.check` runs briefly on 18086.
`werc-site.previous` is stopped and keeps the prior production port mapping.
Container port 8080 is lighttpd; production runs without privileged mode or a
host content bind mount.

## Image and workflow files

`container/Containerfile` builds from Alpine 3.24 with lighttpd, plan9port, and
gawk, copies the pinned werc submodule, content, and assets, and runs as container
UID/GID 1000. `container/lighttpd.conf` defines CGI, rewriting, aliases, and path
protection. `container/initrc.local` chooses Plan 9 paths and the AWK formatter.
`.github/workflows/container.yml` defines rootless CI and GHCR publishing.
`.dockerignore` excludes host code, Git metadata, private config, and common secrets.

Inside the image, `/srv/werc` contains the application and
`/srv/werc/sites/arthurgusmao.com.br` contains pages and templates.
`/pub/meta/revision.txt` exposes the built SHA. The same value appears in the
`org.opencontainers.image.revision` label. Host and container user IDs need not match.

## Commands

    doas rc-service werc-site start
    doas rc-service werc-site verify
    doas rc-service werc-site stop
    doas su-exec werc env HOME=/var/lib/werc werc-deploy
    doas su-exec werc env HOME=/var/lib/werc werc-deploy rollback
    doas rc-service caddy checkconfig
    doas rc-service cloudflared restart
    doas rc-update show

System logs contain the `werc-deploy` tag. Tunnel logs live at
`/var/log/cloudflared.log` and `/var/log/cloudflared.err`. Use `werc-podman logs`
for application stderr. Custom log rotation is a separate host policy.

External state includes GitHub settings, GHCR visibility/retention, DNS, and the
Cloudflare tunnel route. The installer does not recreate those account settings.
