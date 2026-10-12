# Design decisions

[Server guide](/notes/server/)

## Simplicity takes priority

The server uses one running production container, one deployment script, one
service user, one cron job, and a fixed Caddy proxy. The code is ordinary POSIX
shell with explicit command arguments and cleanup. No framework or generated
host program sits between the reader and the actual operations.

An update tests a temporary container, stops production, and starts its replacement.
The previous production container stays stopped for rollback. The tradeoff is a
brief restart outage. That is easier to understand than maintaining two active
slots, proxy promotion, synchronized marker files, and separate transaction locks.

## What was removed

The old host lighttpd, webhook secret/service, file-copy deploy scripts, duplicate
site checkout, and personal-user site containers were removed. lighttpd remains
inside the website image. Later, blue/green staging and promotion helpers, the
active-slot state, changing Caddy upstream, and privileged root deployment cron
were replaced by the single rootless update program.

The service runtime stays independent of desktop `/run/user` mounts. The fixed
storage paths are preserved because changing an established Podman database's
paths would create another migration problem rather than improve readability.

## Important lessons retained

BusyBox cron needs its directory timestamp updated after direct file edits.
A single-architecture image should be pulled as an image rather than inspected
as a manifest list by the installed Podman. A deployment lock must not be inherited
by long-lived conmon processes. These fixes remain in the smaller implementation.

PRs build and test on GitHub-hosted runners, and only master publishes the SHA
image. The server pulls public images; it does not need GitHub credentials or
execute downloaded host deployment scripts.

## Verification and limits

Test first installation, normal replacement, rejected candidates, failed
production startup, rollback, interrupted-rename recovery, and service startup.
Check installed scripts against source and verify the exact public revision
after an automatic deployment. Tests of one script are not evidence of a full
notebook reboot; reboot recovery still needs an explicit real-machine test.

The notebook remains a single host that must stay awake, powered, and connected.
No container arrangement removes that constraint. A stopped previous container
is one rollback release, not a disaster-recovery backup. Running but unhealthy
applications require diagnosis; custom log retention is not installed here.

CI runners, Alpine tags, and package indexes can change. The workflow builds its
native architecture, not a multi-architecture release. Host-script installation,
Cloudflare settings, branch protection, registry visibility, and private credential
backups remain explicit administrative tasks.
