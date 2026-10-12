# Troubleshooting

[Server guide](/notes/server/)

Diagnose the request path from inside outward: container, fixed Caddy upstream,
tunnel, and public hostname. A connected tunnel does not establish origin health.

## Site unavailable

    doas rc-service werc-site verify
    curl -fsS -H 'Host: arthurgusmao.com.br' http://127.0.0.1:18085/pub/meta/revision.txt
    curl -fsS -H 'Host: arthurgusmao.com.br' http://127.0.0.1:8080/pub/meta/revision.txt
    doas rc-service caddy status
    doas rc-service cloudflared status

If the container fails, read its logs and inspect its state. If the container
works but Caddy fails, check `/etc/caddy/Caddyfile`. It should always target
18085. If local Caddy works but the public site fails, inspect the hostname route,
DNS, network connection, and tunnel stderr.
[Cloudflare documents origin troubleshooting](https://developers.cloudflare.com/tunnel/troubleshooting/).

## A change is not live

    doas cat /etc/crontabs/werc
    doas test -e /etc/werc/paused
    doas rc-service crond status
    doas tail -80 /var/log/messages
    doas su-exec werc env HOME=/var/lib/werc werc-deploy

Check the master CI run and its publish step. Compare its SHA with the public
marker. A successful `test -e` means the scheduler is paused. Image availability,
download time, validation failures, and a newer master can delay replacement.
When editing cron directly, touch `/etc/crontabs` so BusyBox reloads it promptly.

## Podman sees a different system

    doas su-exec werc env HOME=/var/lib/werc werc-podman ps -a
    doas su-exec werc env HOME=/var/lib/werc werc-podman info

Use the service identity and wrapper. Changing only `XDG_RUNTIME_DIR` does not
change a database that already records other paths. Rootful and personal-user
Podman are not this service. The wrapper fixes all three relevant paths.

## Port conflict or stalled update

    doas ss -lntp
    ps -eo user,pid,comm

Production uses 18085; the temporary check uses 18086. Identify the actual owner
before stopping anything. Do not stop production to free the test port.
A leftover test container is removed by the next exclusive deployment attempt.
The parent flock releases when its process exits and is not inherited by conmon.

## Candidate rejected

Check the full SHA, OCI label, revision marker, and container stderr. Test with
`Host: arthurgusmao.com.br`. The image needs executable werc, Plan 9 rc, the
correct formatter, the core pages/assets, and private-path protection.
A failed candidate must leave production untouched.

## Public revision or styling looks stale

Compare direct container, local Caddy, and public markers. Check whether the
tunnel points to this machine and whether external caching rules serve old data.
The stylesheet link contains a version query; update it when changing CSS.
A healthy local deployment does not guarantee an externally healthy tunnel.

## Laptop and storage

    df -h /var/lib/werc
    doas su-exec werc env HOME=/var/lib/werc werc-podman images

Watch disk usage and log sizes. The notebook must remain powered, awake, and
connected. The installer does not change lid-close or desktop power policy.
User cron retries updates after network recovery; OpenRC restores production
from local state at boot. No setup here creates a second independent host.
