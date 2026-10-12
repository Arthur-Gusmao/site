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

Rootless container services use a dedicated runtime at `/run/werc-oci`,
managed by OpenRC rather than by graphical login sessions. To inspect
production containers from a shell, use:

```sh
XDG_RUNTIME_DIR=/run/werc-oci podman ps -a
```

Do not manage production containers with the graphical session's default
`/run/user/1000` runtime.

## License

Original website code and content: MIT License.

Oxanium: SIL Open Font License 1.1.
werc: Public domain.

See the respective license files and upstream projects.
