# Site server

    Cloudflare Tunnel -> container (lighttpd + werc)
    GitHub -> tested image -> automatic update

One rootless container on Alpine Linux. No database or extra reverse proxy.
The tunnel connects directly to `http://127.0.0.1:8080`.

## Install

    git clone --recurse-submodules https://github.com/Arthur-Gusmao/site.git
    cd site
    doas sh ops/werc-install

Save your tunnel token in `/etc/cloudflared/token` (root-owned, mode `0600`).
Set the Cloudflare hostname route to `http://127.0.0.1:8080`.

    doas werc update
    doas werc start

## Use

    doas werc status
    doas werc logs
    doas werc update
    doas werc pause
    doas werc rollback
    doas werc resume

`rollback` restores the previous release and pauses automatic updates.
`update SHA` deploys a specific full commit SHA, even while paused.

## Edit and publish

Edit pages in `site/` and appearance in `pub/style/arthur.css`.
Merge to `master`: CI builds, checks and publishes the image; the server
checks for updates each minute. New images are tested before replacement.
A failed replacement restores the previous container. Updates cause a brief outage.
Rerun the installer to update host scripts.

## Details

- [Development](/notes/server/development/)
- [Installation](/notes/server/installation/)
- [Architecture](/notes/server/architecture/)
- [CI/CD](/notes/server/cicd/)
- [Operations](/notes/server/operations/)
- [Recovery](/notes/server/recovery/)
- [Troubleshooting](/notes/server/troubleshooting/)
- [File reference](/notes/server/reference/)
- [Design decisions](/notes/server/decisions/)
- [Audit](/notes/server/audit/)
