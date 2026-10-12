# Site server

One running container, a fixed Caddy upstream, and one rootless deployment script.
Updates have a brief restart window; failed replacements restore the previous image.

Read the [audit guide](https://arthurgusmao.com.br/notes/server/audit/) first.
The [complete documentation](https://arthurgusmao.com.br/notes/server/) covers setup,
operation, recovery, and the explicit tradeoffs.

## Install

```sh
git clone --recurse-submodules https://github.com/Arthur-Gusmao/site.git
cd site
doas sh ops/werc-install
```

Configure the Cloudflare hostname route to `http://127.0.0.1:8080` and save its
token privately in `/etc/cloudflared/token`, root-owned mode 0600.
The installer owns the single-site Caddyfile and dedicated werc user crontab.

```sh
doas rc-service caddy start
doas rc-service cloudflared start
doas su-exec werc env HOME=/var/lib/werc werc-deploy
doas rc-service werc-site start
```

## Operate

```sh
doas rc-service werc-site verify
doas su-exec werc env HOME=/var/lib/werc werc-podman ps -a
curl -fsS https://arthurgusmao.com.br/pub/meta/revision.txt
doas tail -50 /var/log/messages
```

`werc-site` serves on 18085. `werc-site.check` is a temporary test on 18086.
`werc-site.previous` is stopped and retains the prior release for rollback.
Production storage and runtime are fixed by `werc-podman` and are independent
of personal Podman storage and graphical login.

## Rollback

Pause automatic updates, wait for any active transaction to finish, then:

```sh
doas touch /etc/werc/paused
doas su-exec werc env HOME=/var/lib/werc werc-deploy rollback
```

Explicit SHA and rollback commands ignore the pause file. Remove the file to
resume following master. Reinstall reviewed source to update host scripts;
publishing an image updates only the website. Never use `system reset` for cleanup.
