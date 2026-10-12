# Reproduce the server

[Server guide](/notes/server/)

## Prerequisites

Use an Alpine Linux machine with OpenRC, working network access, and authorized
root access through doas or a root shell. The first checkout requires Git.
The container filesystem must support the Podman storage driver; this notebook
uses a local ext4 filesystem. The site image is currently built on GitHub's
Ubuntu runner for its native architecture; the tested production target is x86-64.
The cloudflared installer supports x86-64 and ARM64, which does not itself
make the website image multi-architecture.

A Cloudflare-managed domain, a configured tunnel, and a public GHCR package
are external prerequisites. The installer does not create Cloudflare accounts,
DNS records, GitHub repositories, branch protection, or registry visibility.

## Get the complete source

    git clone --recurse-submodules https://github.com/Arthur-Gusmao/site.git
    cd site
    git submodule update --init --recursive
    doas sh ops/werc-oci-install

The installer installs `podman`, `shadow-subids`, `passt`, `curl`, `git`,
`flock`, `su-exec`, and `caddy`. When cloudflared is absent, it downloads the
official `2026.10.0` release and verifies the architecture-specific SHA-256
stored in the script. An existing cloudflared binary is preserved.

It creates or reuses `werc`, allocates missing subordinate-ID mappings,
creates the service directories, installs root-owned scripts, and enables
`cgroups`, `werc-oci`, `caddy`, `cloudflared`, and `crond` in the default runlevel.
It installs one cron entry and touches the crontab directory so BusyBox notices
the schedule change. Existing unrelated cron entries remain intact.

## Configure the tunnel

Create the published hostname route for `arthurgusmao.com.br` with service
`http://127.0.0.1:8080`. Place the tunnel token in `/etc/cloudflared/token`
using a local editor or another private transfer method. Do not put its value
in the repository, shell examples, public notes, or command arguments.

    doas chmod 0600 /etc/cloudflared/token
    doas chown root:root /etc/cloudflared/token

The token directory is root-owned with mode `0700`. cloudflared reads the file
through `--token-file`. Cloudflare holds the remotely managed route configuration.
Refer to [the official tunnel setup guide](https://developers.cloudflare.com/tunnel/get-started/).

## First deployment

    doas rc-service caddy start
    doas rc-service cloudflared start
    doas werc-oci-deploy
    doas rc-service werc-oci start
    doas rc-service werc-oci verify
    curl -fsS https://arthurgusmao.com.br/pub/meta/revision.txt

A new installation records `active-slot=none`. Its initial Caddy configuration
points to port `18085`, but no application is serving yet. The first candidate
is staged in green and promoted only after checks succeed. There is no legacy
web server or bootstrap port. Verification reports failure until a release exists.

The tunnel must be connected for promotion's public-domain checks to pass.
The image tagged with the current `master` SHA must exist and be anonymously
pullable from GHCR. If it is not available, the check waits for a later cycle.

## Repeat installation

Running the installer again preserves the active slot and existing Caddyfile.
It replaces the installed scripts and stores their previous versions in one
`/etc/werc-oci/previous-install` directory. It does not start a duplicate cron
job or automatically restart an already running cloudflared process.

After changing its service definition, restart cloudflared deliberately:

    doas rc-service cloudflared restart

The installer is specific to this website. On a machine with an existing Caddy
site but no site deployment state, first installation backs up that Caddyfile
and installs this site's configuration. Review the script before using a shared
host. Package repositories and the Alpine base tag can change, so reproduction
means the same configuration and behavior, not byte-identical builds forever.
