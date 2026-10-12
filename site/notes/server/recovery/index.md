# Rollback and recovery

[Server guide](/notes/server/)

## Restore the last release

Pause the automatic checks, let any transaction finish, and inspect production
and its stopped backup:

    doas touch /etc/werc/paused
    doas su-exec werc env HOME=/var/lib/werc werc-podman ps -a
    doas su-exec werc env HOME=/var/lib/werc werc-deploy rollback
    doas rc-service werc-site verify
    curl -fsS https://arthurgusmao.com.br/pub/meta/revision.txt

Rollback reads the previous container's exact image ID, tests it on the temporary
port, and uses the normal replacement procedure. The former production release
then becomes the stopped backup. Resume automatic deployment by removing the
pause file when you want master to be current again.

## Interrupted updates

If production is missing and `werc-site.previous` exists, startup renames the
backup to production and starts it. Both OpenRC and the deploy script handle
this case without consulting GitHub.

If production exists but is stopped, startup starts it. A running but unhealthy
container is not automatically rolled back by the next matching-SHA check;
inspect logs and use explicit rollback if a valid backup remains.

A leftover `werc-site.check` is only a test container. The next deployment removes
it under its exclusive lock before testing. Do not manually remove containers
while another deployment is running. The lock file's existence alone does not
mean a process owns the lock; inspect running processes first.

The replacement exit trap handles normal errors and handled signals. An abrupt
kill can bypass it, which is why startup also recognizes a missing production
container with an existing backup.

## Missing container and no backup

Pause deployment, inspect the local images, and deploy a trusted known-good SHA
explicitly. If network access is unavailable, inspect the exact local image ID
and recreate the container with its fixed name and port:

    doas su-exec werc env HOME=/var/lib/werc werc-podman run -d --name werc-site -p 127.0.0.1:18085:8080 TRUSTED_LOCAL_IMAGE_ID
    doas rc-service werc-site verify

Confirm the name and port are free before using this recovery example.
Do not invent a marker file or change Caddy to match an arbitrary new port.

## Backups and rebuilding

Git preserves source and the pinned submodule. GHCR preserves published images
subject to registry retention. The stopped previous container is one release of
rollback, not an off-machine backup or a complete release history.

The installer retains one previous script/configuration set in
`/etc/werc/previous-install`. Private migration archives are kept under
`/var/backups`; their contents are not served or published. A pre-cleanup archive
is a recovery aid, not an ongoing backup schedule.

Back up the tunnel credential privately and off the notebook when disaster
recovery matters. Rebuild on another host with the installation guide, restore
the credential, and deploy a trusted image. Do not copy `/run` or restore Podman's
internal runtime across machines. The application has no database or content volume.

If the old tunnel connector still runs, Cloudflare may send traffic to both
machines until the old connector is stopped.
