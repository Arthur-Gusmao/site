# Architecture

[Server guide](/notes/server/)

## Request path

Cloudflare sends requests for `arthurgusmao.com.br` through the tunnel to
`http://127.0.0.1:8080`. Caddy always proxies to `127.0.0.1:18085`.
The rootless container runs lighttpd on port 8080. lighttpd serves public assets
and executes werc to render Markdown as HTML.

    :8080 {
        bind 127.0.0.1
        reverse_proxy 127.0.0.1:18085
    }

Caddy does not change during deployment. Public HTTPS is handled by Cloudflare;
the local origin speaks HTTP. The application ports bind only to loopback.
The tunnel route is configured externally; see [Cloudflare's guide](https://developers.cloudflare.com/tunnel/get-started/).

## Three container names, one running production service

- `werc-site`: production, always on host port 18085.
- `werc-site.check`: temporary candidate, on host port 18086 while testing.
- `werc-site.previous`: stopped prior production container, retained for rollback.

There are no blue/green slots, active-slot file, changing upstream, or promotion
command. The stopped backup keeps its original production port mapping.
After candidate testing, the old production container is stopped and renamed;
the new container takes its normal name and port. This creates a brief outage.

## Identities

The host user `werc` owns Podman storage and executes deployment from its own
crontab. Its home is `/var/lib/werc`; it has no interactive login shell.
Root installs scripts, configures cron, and controls OpenRC. Caddy uses its
packaged service identity. cloudflared reads a private root-owned token.
Routine image deployment does not need root or access to Caddy's admin endpoint.

Rootless namespaces use mappings in `/etc/subuid` and `/etc/subgid`.
The installer allocates missing 65,536-ID ranges after existing allocations.
[Podman documents the rootless requirements](https://docs.podman.io/en/stable/markdown/podman.1.html).

## Storage and boot

`werc-podman` fixes the persistent store at
`/var/lib/werc/.local/share/werc-oci/storage`, runroot at `/run/werc-oci/storage`,
and temporary runtime at `/run/werc-oci/libpod`. These existing paths are kept
to avoid changing an established Podman database. They are independent of a
personal user's storage and graphical-login mounts.

OpenRC creates `/run/werc-oci` as `werc`, mode 0700, before starting the container.
The `werc-site` service needs local mounts and cgroups and starts before Caddy.
It can restore an interrupted rename, start production, and retry local revision
verification ten times. It uses local images without querying GitHub.

cloudflared starts after Caddy, needs networking, and is supervised with a
five-second restart delay. User cron also starts a stopped production container
before checking for updates. A running but unhealthy application still requires
diagnosis; the scheduler does not implement a general health supervisor.

The site and assets are inside the image. There is no host content mount,
application database, or persistent application volume.
