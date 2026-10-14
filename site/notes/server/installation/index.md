# Installation

[Server guide](/notes/server/) contains the fresh-install commands.
The installer requires Alpine Linux, OpenRC and root via `doas`.
It installs Podman, a dedicated user, services and the update cron job.
It downloads a pinned, checksum-verified cloudflared binary if missing.
Port `8080` must be available for the website container.

## Update host scripts

Wait for any deployment to finish, then pause updates and reinstall:

    doas werc pause
    doas sh ops/werc-install
    doas werc update FULL_COMMIT_SHA
    doas werc resume

Replaced host scripts are backed up in `/etc/werc/previous-install`.
The installer owns the dedicated `werc` crontab.
