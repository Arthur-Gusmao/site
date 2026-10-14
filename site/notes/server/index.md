# Site server

    Cloudflare Tunnel -> container (lighttpd + werc)
    GitHub -> CI -> image -> automatic update

One rootless container on Alpine/OpenRC, listening on `127.0.0.1:8080`.
Markdown pages, Oxanium, no database or extra proxy.

## Run on another machine

Install Git and Docker or Podman (Linux containers), then:

    git clone --recurse-submodules https://github.com/Arthur-Gusmao/site.git
    cd site
    ./run

Open `http://localhost:8080`. Stop with Ctrl+C; run `./run 8081` to use another
port. This builds the checked-out source and requires no tunnel token.
Check out a specific commit first to reproduce that source version.

## Install an always-on server

This optional setup requires Alpine/OpenRC and an x86_64 host for the
images published by this repository's CI. A public hostname requires
your own Cloudflare Tunnel token and route.

    git clone --recurse-submodules https://github.com/Arthur-Gusmao/site.git
    cd site
    doas sh ops/werc-install

Save the tunnel token in `/etc/cloudflared/token`, root-owned, mode `0600`.
Set the Cloudflare hostname route to `http://127.0.0.1:8080`.

    doas werc update
    doas werc start

## Operate

    doas werc status
    doas werc logs
    doas werc update
    doas werc restart
    doas werc stop
    doas werc start

Updates follow `master` each minute. CI tests the image before publication.
The server replaces the container and checks its HTTP revision, pages,
stylesheets, font and private-path protection. If the check fails, it
restores the old container. Updates briefly interrupt requests.

## Restore a release

    doas werc rollback

Rollback restores the previous container and pauses automatic updates.
To deploy a known full commit SHA, or resume automatic updates:

    doas werc update FULL_COMMIT_SHA
    doas werc resume

## Update host scripts

Wait for any active deployment to finish, then:

    doas werc pause
    doas sh ops/werc-install
    doas werc update FULL_COMMIT_SHA
    doas werc resume

Image updates change the website; host scripts require reinstalling.

## Edit and preview

Pages live in `site/`; appearance in `pub/style/style.css`.
Navigation and footer live in `site/_werc/lib/`. Use indented code blocks
in Markdown. The formatter uses Plan 9 AWK.

    ./run

Stop with Ctrl+C and rerun after editing. The local script uses your own
Docker or Podman. Production uses the dedicated `werc` user, storage in
`/var/lib/werc/.local/share/werc-oci/storage` and runtime in `/run/werc-oci`.

## Diagnose

    doas werc status
    doas werc logs
    doas tail -50 /var/log/messages
    curl -fsS http://127.0.0.1:8080/pub/meta/revision.txt

If local HTTP works but the public site fails:

    doas rc-service cloudflared status
    doas tail -50 /var/log/cloudflared.err

Check the tunnel route, token, free port `8080`, published image and pause
flag (`/etc/werc/paused`). Do not reset production Podman storage: it holds
the rollback release. To rebuild the host, install from Git, restore your
private tunnel token and deploy a known commit.
