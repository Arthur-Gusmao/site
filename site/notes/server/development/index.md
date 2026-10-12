# Website development

[Server guide](/notes/server/)

## Repository layout

`site/` contains Markdown pages and site template overrides. `pub/` contains
CSS and the Oxanium font. `vendor/werc` is a pinned Git submodule from
[szhilkin/sz-werc](https://github.com/szhilkin/sz-werc).
`container/` defines the runtime image. `ops/` defines host operations, and
`.github/workflows/container.yml` defines CI.

A page such as `site/notes/server/architecture/index.md` is served at
`/notes/server/architecture/`. A directory's `index.md` is its landing page.
The documentation uses normal links and indented code blocks because this
werc formatter does not implement GitHub-style fenced code blocks.

## Rendering and presentation

The image installs lighttpd, plan9port, and gawk. It rewrites the CGI interpreter
path to `/usr/lib/plan9/bin/rc` and uses `container/initrc.local` to choose the
AWK Markdown formatter. The configured PATH selects plan9port's AWK;
use `/usr/lib/plan9/bin/awk` when checking Markdown on this host.
BusyBox AWK can render this formatter's link expressions differently.
It runs lighttpd in the foreground as container UID/GID
`1000:1000`; the host service user's numeric identity need not match that UID.

`site/_werc/lib/headers.inc` includes the viewport setting and
`/pub/style/arthur.css`. `top_bar.inc` defines navigation, and `footer.inc`
defines the footer. Oxanium is loaded from `/pub/fonts/oxanium.woff2`.
The stylesheet supports narrow screens and light/dark color preferences.
Its URL includes a version query; change that value when changing CSS so
previously cached assets do not hide the update.

lighttpd aliases `/pub/` to the shared assets directory. Requests for virtual
pages go through werc; real static files can be served directly. Direct access
to `_werc` and `.git` path segments is denied.

## Local build

Use personal rootless Podman storage for development, not the production wrapper:

    git submodule update --init --recursive
    revision=$(git rev-parse HEAD)
    podman build --build-arg "REVISION=$revision" -f container/Containerfile -t localhost/site:dev .
    podman run --rm --name site-dev -p 127.0.0.1:18083:8080 localhost/site:dev

In another terminal:

    curl -fsS -H 'Host: arthurgusmao.com.br' http://127.0.0.1:18083/
    curl -fsS -H 'Host: arthurgusmao.com.br' http://127.0.0.1:18083/notes/server/

The foreground container is removed when stopped. Stop it before reusing its
name or port. A local build can include uncommitted files even though its marker
uses `HEAD`; do not treat that marker alone as evidence of a clean build.
CI starts from the checked-out commit and uses its own revision.
Do not use the production test port 18086 for unrelated local development.

## Change and review

Edit content or assets, build or inspect the affected pages, commit the changes,
and open a PR. The existing workflow checks rootless execution and core HTTP
behavior. Check new documentation routes and links as well; CI's core route
list is not an exhaustive test of every Markdown page.

The `.dockerignore` excludes Git metadata, workflows, host scripts, private
werc configuration, environment files, key files, and PEM files. It is not a
substitute for reviewing public content. Anything copied into `site/` or `pub/`
may be published. Keep credentials out of both directories.

Original content is MIT-licensed, Oxanium uses SIL OFL 1.1, and werc is public
domain. See the license files in the repository.
