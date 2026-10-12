# Portable site server

A small website server on an Alpine Linux notebook, with plain shell scripts
that can be read from beginning to end. All documentation is in English.

    Browser -> Cloudflare Tunnel -> Caddy -> one rootless website container
    GitHub -> CI -> GHCR -> user cron -> one deployment script

## Read the documentation

- [Audit guide](/notes/server/audit/): the shortest path through the actual code.
- [Architecture](/notes/server/architecture/): components, identities, ports, and storage.
- [Installation](/notes/server/installation/): reproduce the server and start the tunnel.
- [CI/CD](/notes/server/cicd/): PR tests, publishing, and deployment.
- [Operations](/notes/server/operations/): status, logs, manual deployment, and maintenance.
- [Development](/notes/server/development/): edit pages, templates, and scripts.
- [Recovery](/notes/server/recovery/): rollback, interrupted updates, and rebuilding.
- [Troubleshooting](/notes/server/troubleshooting/): diagnose the request path.
- [Reference](/notes/server/reference/): every operational file and command.
- [Design decisions](/notes/server/decisions/): what was removed and the tradeoffs.

## Everyday workflow

Edit the repository, open a PR, and wait for `CI / image`. Merge to `master`
to publish an image. The notebook checks once per minute, tests a temporary
candidate, and restarts the production container with the new image.
Updates have a brief restart window. A failed replacement restores the old container.

Host-script changes require running the installer from a reviewed checkout.
Publishing an image changes the website, not the installed host scripts.
The [repository](https://github.com/Arthur-Gusmao/site) is the source of truth.

The inspected notebook uses Alpine edge, OpenRC, Podman, Caddy, and cloudflared.
The website image starts from Alpine 3.24. Package versions can change, so this
is a reproducible configuration and procedure, not a byte-identical build promise.
Private credentials and recovery-archive contents are not published here.
