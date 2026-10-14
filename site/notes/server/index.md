# Site server

    Browser -> site container (lighttpd + werc)
    Optional public access: Cloudflare Tunnel -> site container

One `compose.yaml`, no custom host installer or deploy scripts.
Git and Podman with a Compose provider are the host requirements.
On Alpine, install them with `doas apk add git podman podman-compose nftables`.

## Run

    git clone --recurse-submodules https://github.com/Arthur-Gusmao/site.git
    cd site
    podman compose up -d --build

Open `http://localhost:8080`.

## Update and stop

    git pull --ff-only --recurse-submodules
    podman compose up -d --build
    podman compose logs -f
    podman compose down

Updates are manual. CI builds and tests changes without deploying them.
To return to a version, check out its commit and rebuild. Updates can
interrupt requests; there is no automatic rollback. After reboot, run
`podman compose up -d`.

## Public access

Run `cp .env.example .env` and set your private `TUNNEL_TOKEN`.
Configure the Cloudflare hostname route to `http://site:8080`.
The tunnel reaches the site over the Compose network.

    podman compose --profile tunnel up -d --build
    podman compose --profile tunnel logs -f
    podman compose --profile tunnel down

Use a fresh token and hostname route for a separate public deployment.
Keep `.env` out of Git. Set `PORT` there if `8080` is already occupied.
The default setup runs locally without credentials.

## Edit and check

Pages live in `site/`; style in `pub/style/style.css`; templates in
`site/_werc/lib/`. Rebuild after editing. Check HTTP with:

    ./check

The check needs curl. It verifies pages, CSS, Oxanium and private-path denial.

## Moving from the previous setup

Stop the old deployment cron and OpenRC services before starting Compose
on the same host. They are not managed or removed by Compose.
The public tunnel route changes from `http://127.0.0.1:8080` to
`http://site:8080` because the tunnel now runs inside the Compose network.
Keep the old containers until the new setup works; they are separate from
this Compose project.
