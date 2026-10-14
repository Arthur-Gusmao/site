# File reference

[Server guide](/notes/server/)

- `site/`: Markdown pages and layout templates.
- `pub/style/arthur.css`: site appearance, on top of the base `style.css`.
- `vendor/werc/`: pinned rendering engine.
- `container/`: image and lighttpd configuration.
- `.github/workflows/container.yml`: build, test and publish.
- `ops/werc-install`: install host dependencies and services.
- `ops/werc`: commands, HTTP checks and fixed rootless storage.
- `ops/werc-deploy`: test, replace and restore containers.
- `ops/werc-site-openrc`: container lifecycle at boot.
- `ops/cloudflared-openrc`: tunnel lifecycle.

The tunnel token lives in `/etc/cloudflared/token`. The pause flag is
`/etc/werc/paused`. Host-script backups are in `/etc/werc/previous-install`.
