# Troubleshooting

[Server guide](/notes/server/)

Check the container first:

    doas werc status
    doas werc logs
    curl -fsS http://127.0.0.1:8080/pub/meta/revision.txt

If local HTTP works but the public site fails, inspect the tunnel:

    doas rc-service cloudflared status
    doas tail -50 /var/log/cloudflared.err

Its hostname route must target `http://127.0.0.1:8080`.

For failed updates, inspect `/var/log/messages` for `werc-deploy`.
Check whether updates are paused (`/etc/werc/paused`), the image was published,
and port `8080` is available. Host-script changes need the installer.
