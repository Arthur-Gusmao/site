# Architecture

[Server guide](/notes/server/)

## Request path

A visitor connects to `https://arthurgusmao.com.br`. Cloudflare forwards the
request through the connected tunnel to Caddy at `http://127.0.0.1:8080`.
Caddy sends it to the selected rootless container. Inside that container,
lighttpd serves static assets and executes werc's `werc.rc` CGI for virtual
pages. werc renders Markdown using its AWK formatter.

The tunnel's local service speaks HTTP. Public HTTPS and the encrypted tunnel
connection are handled separately. Caddy does not issue certificates for this
loopback listener. The application ports are not bound to the LAN interface.
See [Cloudflare's routing documentation](https://developers.cloudflare.com/tunnel/get-started/).

## Components

- Alpine supplies the operating system and package manager.
- OpenRC orders startup and provides explicit service commands.
- Podman creates containers without a privileged container daemon.
- `werc` is the host service user, with home `/var/lib/werc` and no login shell.
- Caddy runs as its packaged `caddy` user and selects the production upstream.
- cloudflared runs through OpenRC's supervisor, using a root-readable token file.
- BusyBox crond launches the deployment check once per minute.
- GitHub-hosted Actions build and test changes; GHCR stores the images.

## Ports and slots

- `127.0.0.1:8080`: Caddy's origin listener.
- `127.0.0.1:18085`: host port for `werc-blue`.
- `127.0.0.1:18086`: host port for `werc-green`.
- Container port `8080`: lighttpd, mapped by Podman to its slot's host port.
- `127.0.0.1:18083`: temporary HTTP test port on the CI runner.
- Caddy's default admin endpoint is local port `2019`; reload uses it.

Blue and green are deployment slots, not two simultaneous production replicas.
Only one receives normal traffic. Both may run so the older release can be
promoted quickly. There is no automatic traffic failover to the standby slot.

## Persistent and temporary state

Image layers, container metadata, and local container logs belong to `werc`
and live under `/var/lib/werc/.local/share/werc-oci/storage`.
`werc-podman` always supplies the same storage, runroot, and temporary paths.
Temporary state belongs under `/run/werc-oci`, which OpenRC creates with mode
`0700`. It is rebuilt after reboot.

This avoids `/run/user/UID`, whose desktop-session mounts previously hid
runtime state. The production containers do not require a graphical login.
Personal Podman containers use separate storage.

The image contains the site and public assets. Production uses no host content
bind mount, database, or application volume. Editing the checkout cannot change
the running release until a new image is deployed.

## Boot and crash recovery

The `werc-oci` service needs local mounts and cgroups, and starts before Caddy.
It starts the selected container and verifies its revision, then tries the
standby if one exists. Active readiness allows eight attempts; standby readiness
allows two. These checks use local images and loopback HTTP.

cloudflared starts after Caddy and needs networking. Its supervisor restarts
it after a failure, with a five-second delay and no fixed restart count.
Cron also starts a stopped active container before checking GitHub.
This is process recovery; a running but unhealthy application needs diagnosis.

Rootless user namespaces rely on subordinate IDs in `/etc/subuid` and
`/etc/subgid`. The installer allocates a 65,536-ID range when a mapping is
missing. See [Podman's rootless documentation](https://docs.podman.io/en/stable/markdown/podman.1.html).
