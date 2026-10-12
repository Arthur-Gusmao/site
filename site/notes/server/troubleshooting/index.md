# Troubleshooting

[Server guide](/notes/server/)

Start at the application and work outward: container, Caddy, tunnel, then the
public hostname. A connected tunnel alone does not prove its origin is healthy.

## Public site returns 502 or does not load

    doas rc-service werc-oci verify
    curl -fsS -H 'Host: arthurgusmao.com.br' http://127.0.0.1:8080/pub/meta/revision.txt
    doas rc-service caddy status
    doas rc-service cloudflared status
    doas tail -50 /var/log/cloudflared.err

If container verification fails, inspect container state and logs first.
If the container works but local Caddy fails, inspect its upstream and config.
If local Caddy works but the public domain fails, inspect tunnel connectivity,
the hostname route, DNS, and the configured local service URL.
See [Cloudflare troubleshooting](https://developers.cloudflare.com/tunnel/troubleshooting/).

## A merged change is not live

Check the latest `master` Actions run, confirm its publish step passed, compare
its SHA with `/pub/meta/revision.txt`, and inspect the scheduler:

    doas test -e /etc/werc-oci/paused
    doas cat /etc/crontabs/root
    doas rc-service crond status
    doas tail -80 /var/log/messages
    doas werc-oci-deploy

A successful `test -e` means deployment is paused. Image downloads and tests add
time beyond the minute interval. A public/private registry mismatch, a failed
build, networking, or a newer commit can also delay promotion.

After direct edits to the cron file, touch `/etc/crontabs` or install it through
crontab so BusyBox reloads it promptly. The installer already does this.

## Podman shows no production containers

Use the service identity and wrapper:

    doas su-exec werc env HOME=/var/lib/werc werc-podman ps -a
    doas su-exec werc env HOME=/var/lib/werc werc-podman info

Personal users and root have separate stores. Changing only `XDG_RUNTIME_DIR`
is not sufficient when a database already records different paths. The wrapper
sets storage, runroot, and tmpdir explicitly. Missing `/run/werc-oci` should be
repaired through the service or installer, with owner `werc` and mode `0700`.

## Address already in use

    doas ss -lntp

Identify the process holding `18085`, `18086`, or the local development port.
Do not kill an arbitrary process or stop the active slot to make staging pass.
An old deployment, another dev container, or a previous interrupted attempt
may own the port. Inspect the corresponding container names and state first.

## Deployment silently skips or a lock appears stuck

The outer lock prevents overlapping checks. It is deliberately kept outside
container descendants. Inspect running processes and deployment logs rather
than deleting a lock file just because it exists. Stage and promotion use
separate directory locks; see [interrupted staging](/notes/server/recovery/).

A previous implementation inherited the outer lock into conmon, preventing
future checks even after a deployment finished. The current `flock --close`
invocation fixes that. Reinstall host scripts if an older copy is still running.

## Homepage or revision check fails

Confirm the container starts lighttpd, the expected site exists, and the image
label matches the requested SHA. Test with `Host: arthurgusmao.com.br`.
Read container stderr. A shell syntax pass alone does not establish HTTP health.
The image must include executable werc and Plan 9 rc, and the correct formatter.

For a public revision mismatch, compare the container endpoint and local Caddy
with the public URL. Verify the tunnel targets this notebook and review any
Cloudflare caching rules. Promotion uses a query parameter and a no-cache
request, but it cannot override every externally configured cache policy.

## Resource and power problems

    df -h /var/lib/werc
    doas su-exec werc env HOME=/var/lib/werc werc-podman images

Clean unused images deliberately and watch log sizes. A laptop that sleeps,
loses power, or loses its network cannot serve this site. Lid-close and desktop
power settings are not changed by the installer. After connectivity returns,
cloudflared supervision and the deployment check can recover their own services.
