# Audit

[Server guide](/notes/server/)

Read these files in order:

1. `container/Containerfile`: what runs in the image.
2. `container/lighttpd.conf`: public assets, rendering and private-path denial.
3. `.github/workflows/container.yml`: checks before image publishing.
4. `ops/werc-deploy`: deployment lock, candidate checks and rollback.
5. `ops/werc-install`: host changes, user, services and cron.
6. `ops/werc`: everyday command interface.

Validate shell syntax locally:

    sh -n ops/werc
    sh -n ops/werc-deploy
    sh -n ops/werc-install

Then follow the [development guide](/notes/server/development/) to build
and check HTTP locally. Installation and migration need validation on
an Alpine/OpenRC host with a configured tunnel.
