# Portable site server

This is the complete operating guide for my personal website on an Alpine Linux
notebook. The design uses plain files, small shell scripts, rootless containers,
and existing Unix services. All documentation is in English.

    Browser -> Cloudflare Tunnel -> Caddy -> lighttpd -> werc -> HTML
    GitHub  -> CI tests -> GHCR -> scheduled check -> stage -> promote

## Read the guide

- [Architecture](/notes/server/architecture/): components, users, ports, storage, and boot.
- [Installation](/notes/server/installation/): reproduce the server on another machine.
- [CI/CD](/notes/server/cicd/): pull requests, image publishing, staging, and promotion.
- [Operations](/notes/server/operations/): status, logs, manual deployment, and maintenance.
- [Development](/notes/server/development/): content, templates, local builds, and contributing.
- [Recovery](/notes/server/recovery/): rollback, missing containers, host migration, and backups.
- [Troubleshooting](/notes/server/troubleshooting/): diagnose each layer of the system.
- [Reference](/notes/server/reference/): every operational file, command, and dependency.
- [Design decisions](/notes/server/decisions/): the problems fixed, the cleanup, and practical limits.

## The everyday workflow

Edit the repository, open a pull request, and wait for `CI / image`.
Merge into `master` to build and publish the production image. The notebook
checks for updates every minute, downloads the image, tests the inactive
container, and reloads Caddy to switch traffic. The previous active container
remains available for rollback.

## Scope and source of truth

This guide describes the implementation in
[Arthur-Gusmao/site](https://github.com/Arthur-Gusmao/site), not a generic
container platform. The domain and service paths are specific to this site.
The scripts under `ops/` and the workflow under `.github/workflows/` are the
source of truth. Host-script updates require reinstalling them; publishing
an image updates the website only.

The environment inspected on October 12, 2026 used Alpine edge
`3.25.0_alpha20260805`, OpenRC, Podman `6.1.3`, Caddy `2.11.4`, and cloudflared
`2026.10.0`. These are a dated inventory, not a promise that host packages
will always have those versions. The image starts from `alpine:3.24`.

Credentials, account identifiers from the tunnel token, private configuration,
and recovery-archive contents are deliberately excluded from public documentation.
