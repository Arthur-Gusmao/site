# Site server

```text
PR     -> GitHub Actions -> rootless build and HTTP tests
master -> the same tests -> GHCR image tagged with the commit SHA
server -> check every minute -> stage -> test -> Caddy reload
browser -> Cloudflare Tunnel -> Caddy -> rootless container
```

The `werc` service user owns the containers. Root installs trusted local
scripts and reloads Caddy. Production storage and runtime are independent
of desktop login sessions. The server never runs deployment scripts from
an incoming image or pull request.

## Install on Alpine / OpenRC

```sh
git clone --recurse-submodules https://github.com/Arthur-Gusmao/site.git
cd site
doas sh ops/werc-oci-install
```

The installer creates the service user, allocates subordinate IDs, installs
packages and a verified Cloudflare client, installs services, and schedules
one cron job. Running it again preserves the active slot and Caddy settings.
A single `previous-install` directory holds the previous scripts.
Promotion keeps one `Caddyfile.previous` for recovery; backups do not grow
with every installation or deployment.

Save your Cloudflare Tunnel token as `/etc/cloudflared/token`, owned by root
with mode `0600`. Configure the public hostname `arthurgusmao.com.br` to
forward to `http://127.0.0.1:8080`. Keep the token outside Git.

```sh
doas rc-service caddy start
doas rc-service cloudflared start
doas werc-oci-deploy
doas rc-service werc-oci start
```

A new installation starts with `active-slot=none` until its first successful
deployment. No web server runs on the host except Caddy. The first deployment
requires a working tunnel because promotion verifies the public domain.
The GHCR package must be public; no server-side GitHub credentials are needed.

## Daily use

Open a PR and wait for `CI / image`. Merge it into `master` to publish the
image. The server checks once per minute and deploys when the image is
available. Downloading and validating an image takes additional time.
Failed builds or unavailable images leave production unchanged.

Deployment replaces the inactive slot, tests pages, CSS, the font, the
revision, and private-path protection, then reloads Caddy. The other slot
remains available for rollback. OpenRC restores containers at boot using
local images; cron recovers a stopped active container and checks for updates.

```sh
doas rc-service werc-oci verify
doas su-exec werc env HOME=/var/lib/werc werc-podman ps -a
curl -fsS https://arthurgusmao.com.br/pub/meta/revision.txt
doas tail -50 /var/log/messages
doas tail -50 /var/log/werc-oci.log
```

Use `werc-podman` for production. Storage lives at
`/var/lib/werc/.local/share/werc-oci/storage`; temporary runtime lives at
`/run/werc-oci`. Personal Podman storage is separate.

## Rollback

```sh
doas touch /etc/werc-oci/paused
doas cat /etc/werc-oci/active-slot
# When green is active, blue is the previous release:
doas werc-oci-promote blue
```

Pause automatic deployment before rollback. Remove `/etc/werc-oci/paused`
to resume it. Run `doas rc-service werc-oci restore` to recover stopped
containers. Both slots must remain intact to retain rollback.

Unused images can be removed explicitly:

```sh
doas su-exec werc env HOME=/var/lib/werc werc-podman image prune -a
```

Images used by either slot are retained. Do not use `system reset` for
routine maintenance. Install changes to host scripts by rerunning the
installer; CD updates the website image only.

The notebook must stay powered on, awake, and connected to serve the site.

References: [Podman rootless](https://docs.podman.io/en/stable/markdown/podman.1.html),
[Caddy](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy), and
[Cloudflare Tunnel](https://developers.cloudflare.com/tunnel/get-started/).
