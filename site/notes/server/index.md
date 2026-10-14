# Site server

    Cloudflare Tunnel -> container (lighttpd + werc)
    GitHub -> CI -> image -> automatic update

One rootless container on Alpine/OpenRC, listening on `127.0.0.1:8080`.
Markdown pages, Oxanium, no database or extra proxy.

## Install

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

The installer keeps replaced scripts in `/etc/werc/previous-install`.
Image updates change the website; host scripts require reinstalling.

## Edit and preview

Pages live in `site/`; appearance in `pub/style/arthur.css`.
Navigation and footer live in `site/_werc/lib/`. Use indented code blocks
in Markdown. The formatter uses Plan 9 AWK.

    git submodule update --init --recursive
    revision=$(git rev-parse HEAD)
    podman build --build-arg "REVISION=$revision" -f container/Containerfile -t localhost/site:dev .
    podman run --rm -p 127.0.0.1:18083:8080 localhost/site:dev

Open `http://localhost:18083`. In another terminal, check the site:

    sh ops/werc check http://127.0.0.1:18083 "$(git rev-parse HEAD)"

Use personal Podman for preview; production uses the dedicated `werc` user,
with storage in `/var/lib/werc/.local/share/werc-oci/storage` and runtime in
`/run/werc-oci`. Stop the preview with Ctrl+C. Rebuild after editing.

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
