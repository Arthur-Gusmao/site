# Design decisions

[Server guide](/notes/server/)

The tunnel connects directly to the website container, with no intermediate
proxy service or configuration.

Keep one rootless container and commit-tagged images: releases remain
identifiable and the previous container remains available for rollback.
Keep candidate testing and the deployment lock to avoid publishing a broken
image or running overlapping replacements.

Use `werc` as the host command so everyday operations do not require
remembering the service user, environment or Podman storage flags.

Keep the original appearance, the locally hosted Oxanium font and Markdown pages. Keep the rendering
engine as a pinned submodule. Updates briefly interrupt requests; this is
a small personal website rather than a zero-downtime deployment system.
