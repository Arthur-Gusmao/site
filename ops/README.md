# Site server

    Cloudflare Tunnel -> container (lighttpd + werc)
    GitHub -> tested image -> automatic update

One rootless container, no database, no JavaScript and no extra reverse proxy.
The tunnel points to `http://127.0.0.1:8080`.

## Install on Alpine / OpenRC

```sh
git clone --recurse-submodules https://github.com/Arthur-Gusmao/site.git
cd site
doas sh ops/werc-install
```

Save the Cloudflare tunnel token in `/etc/cloudflared/token`, owned by root
with mode `0600`. Configure its hostname route to `http://127.0.0.1:8080`.

```sh
doas werc update
doas rc-service werc-site start
doas rc-service cloudflared start
```

## Everyday commands

| Task | Command |
| --- | --- |
| Status | `doas werc status` |
| Container logs | `doas werc logs` |
| Update now | `doas werc update` |
| Deploy a full commit SHA | `doas werc update SHA` |
| Pause automatic updates | `doas werc pause` |
| Restore previous release and pause | `doas werc rollback` |
| Resume automatic updates | `doas werc resume` |

Changes to `master` publish a tested image. The server checks each minute.
It tests the new image on port `18086`, then replaces the live container.
If replacement fails, it restores the previous container. Updates briefly
interrupt requests. Host-script changes require rerunning the installer.

## Update host scripts

Wait for an active deployment to finish before reinstalling:

```sh
doas werc pause
doas sh ops/werc-install
doas werc update FULL_COMMIT_SHA
doas werc resume
```

The installer preserves replaced scripts in `/etc/werc/previous-install`.
Port `8080` must be available for the website container.

More details: [server guide](../site/notes/server/index.md).
