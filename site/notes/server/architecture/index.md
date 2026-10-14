# Architecture

[Server guide](/notes/server/)

The request path is Cloudflare Tunnel -> lighttpd -> werc. lighttpd and
werc run in one container. werc renders Markdown without a database.
The tunnel connects directly to the container.

The host runs Alpine/OpenRC. `cloudflared` maintains the tunnel and
`werc-site` starts the container after boot. A dedicated `werc` user runs
rootless Podman; root installs scripts and manages services.

- Live container: `werc-site`, loopback port `8080`.
- Temporary candidate: `werc-site.check`, loopback port `18086`.
- Previous release: `werc-site.previous`, stopped.
- Persistent storage: `/var/lib/werc/.local/share/werc-oci/storage`.
- Runtime and deployment lock: `/run/werc-oci`.

GitHub Actions publishes commit-tagged images to GHCR. A user cron job
checks `master` every minute. Deployments are locked, tested before
replacement, and restore the previous container if replacement fails.
