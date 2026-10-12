# Rollback and recovery

[Server guide](/notes/server/)

## Roll back one release

Pause the scheduler and let any running deployment finish. Check which slot is
active and confirm the other slot is the version you want:

    doas touch /etc/werc-oci/paused
    doas cat /etc/werc-oci/active-slot
    doas su-exec werc env HOME=/var/lib/werc werc-podman ps -a

If green is active and blue holds the previous release:

    doas werc-oci-promote blue
    doas rc-service werc-oci verify
    curl -fsS https://arthurgusmao.com.br/pub/meta/revision.txt

Promotion runs health checks and validates the public revision. Keep the pause
file while investigating. Remove it to resume deployment of current `master`.
A successful stage replaces the inactive slot, so it can replace the older
rollback candidate before promotion; two slots are not a full release archive.

## Recover a stopped or missing container

For an existing stopped container, use:

    doas rc-service werc-oci restore

If the selected container is missing, automatic recovery cannot recreate it
from the state marker alone. Pause deployment, inspect both slots and their
image labels, then either promote an existing healthy slot or reconstruct the
missing one. The container must be named `werc-blue` on port `18085`, or
`werc-green` on port `18086`, using the trusted SHA-tagged image.

An explicit reconstruction example for a missing blue container is:

    doas su-exec werc env HOME=/var/lib/werc werc-podman run -d --name werc-blue -p 127.0.0.1:18085:8080 ghcr.io/arthur-gusmao/site:FULL_40_CHARACTER_SHA
    doas rc-service werc-oci restore

Do this only after confirming that the name and port are free and that the SHA
is the intended trusted release. Verify through the public domain before resuming.

## Configuration mismatch

The state marker and Caddy upstream must agree. Blue means `18085`; green means
`18086`. Do not change the marker alone. A failed promotion normally restores
both. If Caddy reload or filesystem restoration failed, pause the scheduler and
inspect `Caddyfile.previous`, the current file, both containers, and the logs.
Validate a corrected Caddyfile, restore a matching marker, and reload before
running verification. Do not assume the previous file always describes the
currently desired release.

## Interrupted staging

A leftover `werc-blue.previous` or `werc-green.previous` can indicate interruption
outside the shell's cleanup traps. The staging script deliberately refuses to
continue when that backup exists. Inspect the normal name, backup name, running
state, and revisions before removing or renaming anything. Never operate on the
active slot as though it were standby.

Stage and promotion directory locks are cleaned on normal exits and handled
signals. An abrupt kill or power failure can leave a persistent promotion lock.
Confirm that no deployment, stage, or promotion process is running before
removing a stale directory lock. The outer `flock` lock releases when its owning
process exits; the existence of its file alone does not mean it is locked.

## Backups and a new machine

Website source and the pinned submodule are recoverable from Git. Public images
are recoverable from GHCR while retained there. Back up the tunnel credential and
any private host configuration separately, with private permissions and an
off-machine copy when disaster recovery matters.

The installer keeps one previous script set. Promotion keeps one previous
Caddyfile. A one-time private archive was made before the legacy cleanup;
it is not an ongoing backup system, is not served by the website, and must not
be uploaded as public content.

To rebuild elsewhere, follow [installation](/notes/server/installation/), restore
the tunnel credential privately, and deploy the trusted current image. Keep old
connector behavior in mind when moving the same tunnel between machines;
multiple connectors can receive traffic until the old one is stopped.

Rebuilding from source does not require copying Podman's internal database.
The service has no application database or content volume. Do not move its
runtime directories across machines or restore `/run` from a backup.
