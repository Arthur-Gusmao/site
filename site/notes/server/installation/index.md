# Reproduce the server

[Server guide](/notes/server/)

Use Alpine Linux with OpenRC, Git, working network access, and authorized root
access. The tested host is x86-64 with local storage. The workflow builds a native
runner-architecture image; an ARM64 cloudflared binary does not make that image
multi-architecture.

## Install the reviewed source

    git clone --recurse-submodules https://github.com/Arthur-Gusmao/site.git
    cd site
    doas sh ops/werc-install

The installer adds Podman, subordinate-ID helpers, passt, curl, Git, flock,
su-exec, and Caddy. If cloudflared is absent, it downloads the official 2026.10.0
release for x86-64 or ARM64 and checks the SHA-256 stored in the script.
An existing cloudflared binary is preserved.

It creates or reuses `werc`, allocates missing UID/GID mappings, installs
root-owned scripts, creates service directories, and enables cgroups, werc-site,
Caddy, cloudflared, and cron. It installs one user crontab for `werc` and touches
the cron directory so BusyBox notices the change.

The installer owns this single site's Caddy configuration and the dedicated
service user's crontab. Reinstallation restores the configuration from `ops/`;
edit those source files rather than maintaining undocumented host changes.
Existing unrelated root cron jobs are not part of this installer.

## Configure Cloudflare

Create a published hostname route from `arthurgusmao.com.br` to
`http://127.0.0.1:8080`. Save the tunnel token privately in
`/etc/cloudflared/token` using an editor or private transfer. Never put its value
in Git, public documentation, or command-line arguments.

    doas chmod 0600 /etc/cloudflared/token
    doas chown root:root /etc/cloudflared/token

The directory is root-owned, mode 0700. The token stays on the server.
DNS, the tunnel, GHCR package visibility, and GitHub repository settings are
external setup steps. The GHCR package must be publicly pullable.
See [Cloudflare's setup instructions](https://developers.cloudflare.com/tunnel/get-started/).

## Start the site

    doas rc-service caddy start
    doas rc-service cloudflared start
    doas su-exec werc env HOME=/var/lib/werc werc-deploy
    doas rc-service werc-site start
    doas rc-service werc-site verify
    curl -fsS https://arthurgusmao.com.br/pub/meta/revision.txt

Before the first successful deployment, Caddy has no working application upstream
and the service reports that it is waiting for deployment. The deployment tests
the candidate locally; it does not depend on the public tunnel being healthy.
Verify the public domain separately after starting the tunnel.

Running the installer again replaces host scripts and saves one previous copy
under `/etc/werc/previous-install`. It does not automatically restart an already
running tunnel client. Restart cloudflared explicitly after service changes.

Package repositories, runner images, and the Alpine base tag can change. The
procedure reproduces the design and behavior, not every historical binary byte.
