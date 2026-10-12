# arthurgusmao.com.br

Source code for my personal website.

Live website: https://arthurgusmao.com.br

## Philosophy

A small personal website built around Unix principles.

No JavaScript frameworks, databases or unnecessary
dependencies.

## Development

The website uses werc to render Markdown on request.
Production runs from immutable OCI images. Each change to `master` builds
and tests an image before the notebook deploys it. Pull requests run the
same checks without publishing or deploying.

Read the [complete server documentation](https://arthurgusmao.com.br/notes/server/)
for architecture, installation, CI/CD, operations, development and recovery.
The [repository quickstart](ops/README.md) provides the essential commands.

## License

Original website code and content: MIT License.

Oxanium: SIL Open Font License 1.1.
werc: Public domain.

See the respective license files and upstream projects.
