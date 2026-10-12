# arthurgusmao.com.br

Source code for my personal website.

Live website: https://arthurgusmao.com.br

## Philosophy

A small personal website built around Unix principles.

No JavaScript frameworks, databases or unnecessary
dependencies.

## Development

The website uses werc to render Markdown on request.
Production runs from immutable OCI images. Publishing source changes
requires building, staging and promoting a new image; switching slots
does not require restarting Caddy.

## License

Original website code and content: MIT License.

Oxanium: SIL Open Font License 1.1.
werc: Public domain.

See the respective license files and upstream projects.
