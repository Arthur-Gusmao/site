# arthurgusmao.com.br

Source code for my personal website.

Live website: https://arthurgusmao.com.br

## Run on another machine

Requirements: Git and Docker or Podman, running Linux containers.

```sh
git clone --recurse-submodules https://github.com/Arthur-Gusmao/site.git
cd site
./run
```

Open http://localhost:8080. Stop with Ctrl+C.
If the port is busy, use `./run 8081`. The script builds from this checkout,
including local edits. No tunnel token or Alpine host is needed to run locally.

## Edit the site

| Change | File |
| --- | --- |
| Home | `site/index.md` |
| Pages | `site/<page>/index.md` |
| Appearance | `pub/style/style.css` |
| Navigation | `site/_werc/lib/top_bar.inc` |
| Footer | `site/_werc/lib/footer.inc` |

To add `/example/`, create `site/example/index.md` and write Markdown.
Use indented code blocks; the formatter does not support fenced code blocks.

After editing, stop the site and run `./run` again.
To reproduce a particular version, check out that commit before running.
The source revision is pinned by Git; package downloads are not a promise
of byte-identical images.

For an always-on server with automatic updates and a public tunnel,
see the [server guide](site/notes/server/index.md).

## License

Original website code and content: MIT License.

Oxanium: SIL Open Font License 1.1.
werc: Public domain.

See the respective license files and upstream projects.
