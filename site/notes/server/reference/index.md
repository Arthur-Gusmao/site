# Files and commands

[Server guide](/notes/server/)

## Host scripts

- `ops/werc-oci-install`: installs dependencies, users, mappings, scripts, services, and cron.
- `ops/werc-podman`: enforces the service identity and production storage paths.
- `ops/werc-oci-deploy`: checks current master, recovers stopped production, stages, and promotes.
- `ops/werc-oci-stage`: replaces and tests the inactive container, with failure cleanup.
- `ops/werc-oci-promote`: validates a candidate, reloads Caddy, verifies publicly, and rolls back failures.
- `ops/werc-oci-openrc`: restores and verifies slot containers during service startup.
- `ops/cloudflared-openrc`: supervises the tunnel client.
- `ops/Caddyfile`: the initial loopback-only site proxy template.
- `ops/README.md`: a short repository-side operating guide.

Executable host scripts are installed as root-owned mode `0755` files under
`/usr/local/sbin`. Service definitions are installed under `/etc/init.d`.
Review source at [the repository](https://github.com/Arthur-Gusmao/site/tree/master/ops).

## Runtime and state paths

- `/var/lib/werc`: private service-user home, owner `werc`, mode `0700`.
- `/var/lib/werc/.local/share/werc-oci/storage`: persistent production Podman store.
- `/run/werc-oci`: private service-owned runtime directory.
- `/run/werc-oci/storage`: explicit Podman runroot.
- `/run/werc-oci/libpod`: explicit Podman temporary runtime directory.
- `/etc/werc-oci/active-slot`: `none`, `blue`, or `green`; root-owned, mode `0644`.
- `/etc/werc-oci/paused`: presence pauses scheduled deployment and its recovery step.
- `/run/werc-oci-deploy.lock`: outer deployment flock file.
- `/run/werc-oci/werc-oci-stage.lock`: staging directory lock.
- `/etc/werc-oci/promote.lock`: promotion directory lock.
- `/etc/werc-oci/previous-install`: one private copy of previously installed scripts.
- `/etc/werc-oci/Caddyfile.previous`: one prior proxy configuration for recovery.
- `/etc/caddy/Caddyfile`: current origin proxy configuration.
- `/etc/cloudflared/token`: private tunnel credential, root-owned, mode `0600`.
- `/etc/crontabs/root`: root's deployment schedule and other existing jobs.
- `/etc/subuid` and `/etc/subgid`: rootless subordinate identity mappings.

## Image contents

- `container/Containerfile`: Alpine base, packages, copied content, labels, UID, and command.
- `container/lighttpd.conf`: CGI, rewriting, static aliases, port, and private-path blocking.
- `container/initrc.local`: werc's Plan 9 path and AWK formatter.
- `/srv/werc`: werc inside the image; no corresponding production host content tree is needed.
- `/srv/werc/sites/arthurgusmao.com.br`: website Markdown and template overrides.
- `/srv/werc/pub`: public assets and the generated revision marker.
- `/usr/lib/plan9/bin/rc`: werc's CGI interpreter.
- `/pub/meta/revision.txt`: HTTP path exposing the full built commit SHA.
- `org.opencontainers.image.revision`: OCI label used to verify the deployed image.

## Service commands

    doas rc-service werc-oci start
    doas rc-service werc-oci restore
    doas rc-service werc-oci verify
    doas rc-service werc-oci stop
    doas rc-service caddy checkconfig
    doas rc-service caddy reload
    doas rc-service cloudflared restart
    doas rc-update show

Stopping `werc-oci` stops both containers. `restore` recovers existing containers
without intentionally stopping healthy production. `verify` checks the selected
slot locally. `none` is only the initial undeployed state, not a fallback service.

## Logs and external resources

System logs carry the `werc-deploy` tag. Boot diagnostics use
`/var/log/werc-oci.log`; tunnel stdout and stderr use
`/var/log/cloudflared.log` and `/var/log/cloudflared.err`.
Container logs are available through `werc-podman logs`.

External state consists of the GitHub repository and its Actions settings,
GHCR package visibility and retention, the Cloudflare tunnel and hostname route,
and the domain's DNS configuration. These are not reconstructed by shell scripts
alone. Token values, private archives, and credentials are not published here.
