# Design decisions and cleanup

[Server guide](/notes/server/)

## Keep the system understandable

Use one tested image per commit, one scheduled deployment check, one dedicated
container user, and two named slots. Keep host operations in small reviewed shell
scripts and let OpenRC, cron, Podman, Caddy, and Cloudflare do their respective jobs.
There is no JavaScript framework, application database, orchestration cluster,
public deployment webhook, or notebook-hosted GitHub runner.

Two slots add a small amount of state, but allow testing a candidate before
changing traffic and retaining the previous production container. Separate stage
and promotion scripts make the privilege boundary explicit: werc handles
containers; root handles Caddy and the root-owned state marker.

## What was fixed

The earlier runtime lived under the graphical session's `/run/user` tree.
Desktop login mounts could hide the state used at boot, leaving live container
processes while Podman reported a different state. Production now has a dedicated
service user, storage root, runroot, and temporary directory.

CI previously ran only for master pushes. PR events now run the same build and
HTTP checks, with publishing excluded. The old authenticated webhook returned
502 and depended on a separate deployment secret. Pull-based deployment removed
that external endpoint and custom secret requirement.

Operational testing found three additional issues: BusyBox did not promptly
reload a directly edited cron file; a single-architecture image could not be
queried as a manifest list by the installed Podman; and a container supervisor
inherited a deployment lock. The installer now touches the cron directory,
the deploy check pulls the image, and flock closes its descriptor in descendants.
The deploy script also avoids cron's inaccessible `/root` working directory.

## What was removed

The host lighttpd and webhook packages, old webhook service and secret,
file-copy deployment scripts, old personal-user website containers and build
images, duplicate deployment checkout, and host `/srv/werc` content tree were
removed. lighttpd remains inside the application image, where it serves werc.

Legacy slot support, the old origin port `8081`, accumulated Caddy snapshots,
and repeated installation-backup directories were removed from active operations.
Bootstrap now uses `none`, then the first successful green deployment.
One previous script set and one previous Caddyfile remain bounded recovery aids.
A private pre-cleanup archive was retained outside the served website.

## What was actually validated

The rootless image build and HTTP checks passed on a GitHub-hosted PR run and
master runs. Both slot containers passed local health checks. A deliberately
failed stage restored the old inactive container. Stopping an active container
and invoking OpenRC restoration recovered it. A separate Caddy instance verified
promotion from the fresh `none` state without switching production.

The scheduler then published the exact tested master revision through the
public domain, and its lock was available afterward. The cleanup was followed
by repeat installation, service verification, and another automatic deployment.
A full notebook reboot was not performed during those checks.

## Practical limits

This notebook is a single host. It must remain awake, powered, and connected.
Two slots do not provide independent hardware, automatic failover, or a full
backup history. The scheduler recovers a stopped container but does not restart
every running container that develops an application-level fault.

The source and werc revision are tracked in Git. The cloudflared download is
pinned when installed from scratch. Host packages, the Ubuntu runner, the Alpine
base tag, and fetched package indexes can change; builds are not guaranteed
byte-for-byte identical. The workflow builds a native-architecture image rather
than a multi-architecture release.

Host-script updates require explicit installation. Cloudflare route and DNS
settings, required GitHub checks, registry visibility, and credential backups
are external administration tasks. Custom logs need a host retention policy.
These boundaries are part of understanding and operating this small system.
