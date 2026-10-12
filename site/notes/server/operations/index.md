# Daily operations

[Server guide](/notes/server/)

## Status

    doas rc-service werc-site verify
    doas rc-service caddy status
    doas rc-service cloudflared status
    doas su-exec werc env HOME=/var/lib/werc werc-podman ps -a
    curl -fsS https://arthurgusmao.com.br/pub/meta/revision.txt

Service verification compares the local production marker with the container's
image label. It does not verify the tunnel or every page. Use the production
wrapper as `werc`; personal Podman and rootful Podman use other stores.

## Logs

    doas tail -50 /var/log/messages
    doas tail -50 /var/log/cloudflared.log
    doas tail -50 /var/log/cloudflared.err
    doas su-exec werc env HOME=/var/lib/werc werc-podman logs --tail 50 werc-site

Cron output has the `werc-deploy` tag. Service-start errors also appear in the
command's output. Tunnel and container logs need a host retention policy; the
repository does not install a dedicated rotation policy.

## Deploy and pause

    doas su-exec werc env HOME=/var/lib/werc werc-deploy
    doas touch /etc/werc/paused
    doas rm /etc/werc/paused

The first command checks current master. The pause file prevents automatic
checks, including their stopped-container recovery step. It does not cancel an
already running transaction. Let that transaction finish before manual work.
Explicit SHA and rollback commands ignore the pause file deliberately.

## Manual update and rollback

    doas su-exec werc env HOME=/var/lib/werc werc-deploy FULL_40_CHARACTER_SHA

Use a trusted release whose image has passed CI. An explicit SHA forces a
replacement even when the current revision matches; it uses the same candidate
checks and failure cleanup.

    doas touch /etc/werc/paused
    doas su-exec werc env HOME=/var/lib/werc werc-deploy rollback

Rollback tests the stopped previous container's exact local image, then replaces
production. It can work without registry access. Keep the scheduler paused while
investigating; resuming it follows master again.

## Host maintenance

    git pull --ff-only
    git submodule update --init --recursive
    doas sh ops/werc-install

Host scripts change only when installed from a reviewed checkout. Caddy's fixed
configuration and the service user's cron file are managed by the installer.
Starting `werc-site` recovers a stopped or interrupted production container.
Restarting it deliberately interrupts service.

    doas rc-service werc-site start
    doas su-exec werc env HOME=/var/lib/werc werc-podman image prune -a
    df -h /var/lib/werc

Pruning retains images referenced by containers. Keep `werc-site.previous` when
rollback matters. Do not use `system reset` for ordinary cleanup.
Host packages and cloudflared updates are separate maintenance tasks; the tunnel
service disables self-updates and the installer preserves an existing binary.
