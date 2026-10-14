# arthurgusmao.com.br

Source code for my personal website.

Live website: https://arthurgusmao.com.br

## Philosophy

A small personal website built around Unix principles.

No JavaScript frameworks, databases or unnecessary
dependencies.

## Edit the site

| Change | File |
| --- | --- |
| Home | `site/index.md` |
| Pages | `site/<page>/index.md` |
| Appearance | `pub/style/arthur.css` |
| Navigation | `site/_werc/lib/top_bar.inc` |
| Layout | `vendor/werc/lib/default_master.tpl` (werc template) |
| Footer | `site/_werc/lib/footer.inc` |

To add `/example/`, create `site/example/index.md` and write Markdown.
Use indented code blocks; the formatter does not support fenced code blocks.

## Preview locally

```sh
git submodule update --init --recursive
podman build --build-arg REVISION="$(git rev-parse HEAD)" -f container/Containerfile -t localhost/site:dev .
podman run --rm -p 127.0.0.1:18083:8080 localhost/site:dev
```

Open http://localhost:18083. Stop with Ctrl+C and rebuild after editing.

Changes to `master` build and test an image before automatic deployment.
See [server guide](site/notes/server/index.md) for installation and rollback.

## License

Original website code and content: MIT License.

Oxanium: SIL Open Font License 1.1.
werc: Public domain.

See the respective license files and upstream projects.
