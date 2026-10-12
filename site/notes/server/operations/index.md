# Daily operations

[Server guide](/notes/server/)

## Status and revision

    doas rc-service werc-oci verify
    doas rc-service caddy status
    doas rc-service cloudflared status
    doas cat /etc/werc-oci/active-slot
    doas su-exec werc env HOME=/var/lib/werc werc-podman ps -a
    curl -fsS https://arthurgusmao.com.br/pub/meta/revision.txt

`verify` checks the active container process and its local HTTP revision against
the OCI label. It does not verify every page or the public tunnel. Compare the
public marker with GitHub's current production commit when investigating a deploy.

Use `werc-podman` with the service identity for every production container
command. Plain `podman` as a personal user, or `doas podman` as root, points to
another store and cannot reliably describe this service.

## Logs

    doas tail -50 /var/log/messages
    doas tail -50 /var/log/werc-oci.log
    doas tail -50 /var/log/cloudflared.log
    doas tail -50 /var/log/cloudflared.err
    doas su-exec werc env HOME=/var/lib/werc werc-podman logs --tail 50 werc-blue
    doas su-exec werc env HOME=/var/lib/werc werc-podman logs --tail 50 werc-green

Cron output is tagged `werc-deploy` in system logs. OpenRC restoration diagnostics
use `werc-oci.log`. Container stderr contains lighttpd diagnostics. The custom
log files do not have a dedicated rotation policy in this repository; monitor
their size and configure host log rotation if needed. Do not assume bounded
log retention just because configuration backups are bounded.

## Trigger or pause deployment

    doas werc-oci-deploy

This checks current `master`; it does not accept an arbitrary deployment SHA.
No output may mean the active release is already current, deployment is paused,
or another check holds the lock. Inspect the state and logs to distinguish them.

    doas touch /etc/werc-oci/paused
    doas rm /etc/werc-oci/paused

The first command pauses automatic deployment; the second resumes it. Pause
before rollback or manual slot work. A check already in progress is not cancelled
by creating the pause file, so let it finish before changing the containers.

## Manual staging and promotion

Use a full SHA whose image has already passed CI. Inspect the current active
slot, choose the other slot, then run:

    doas su-exec werc env HOME=/var/lib/werc werc-oci-stage green FULL_40_CHARACTER_SHA
    doas werc-oci-promote green

These are examples for a blue-active server. Replace `green` with `blue` when
green is active. Manual promotion is explicit and can select an older revision;
resuming the scheduler will move production back to current `master`.

## Restore, update, and clean

    doas rc-service werc-oci restore

Restore starts and verifies the active container and attempts the existing
standby without stopping production. Restarting the whole service stops both
containers first and can interrupt service; prefer `restore` for recovery.

For host-script changes, review the checkout and rerun the installer. A pushed
website image does not modify `/usr/local/sbin` or `/etc/init.d`.

    git pull --ff-only
    git submodule update --init --recursive
    doas sh ops/werc-oci-install

To reclaim unused service images deliberately:

    doas su-exec werc env HOME=/var/lib/werc werc-podman image prune -a
    df -h /var/lib/werc

Podman retains images referenced by containers. Keep both slot containers when
rollback matters. Do not run `system reset` as routine cleanup. Host-package
upgrades and cloudflared upgrades are separate maintenance tasks; cloudflared's
service disables self-updates and the installer preserves an existing binary.
