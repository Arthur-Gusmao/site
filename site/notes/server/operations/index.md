# Operations

[Server guide](/notes/server/)

    doas werc status
    doas werc logs
    doas werc update
    doas werc pause
    doas werc resume

`status` checks the live HTTP revision and lists containers. `logs` prints
the last 50 lines from lighttpd. Deployment logs are in `/var/log/messages`.

    doas tail -50 /var/log/messages
    doas rc-service werc-site restart
    doas rc-service cloudflared restart

To restore the previous release and pause automatic updates:

    doas werc rollback

Host-script updates require rerunning `ops/werc-install`.
Do not use Podman `system reset`; production storage contains the rollback image.
