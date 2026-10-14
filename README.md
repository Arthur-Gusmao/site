# arthurgusmao.com.br

Markdown + werc + lighttpd in one container. Oxanium, no database or JavaScript.

## Run

Install Git, Podman and a [Compose provider](https://docs.podman.io/en/stable/markdown/podman-compose.1.html) (`podman-compose`). On Alpine:

```sh
doas apk add git podman podman-compose nftables
```

Then:

```sh
git clone --recurse-submodules https://github.com/Arthur-Gusmao/site.git
cd site
podman compose up -d --build
```

Open http://localhost:8080.

## Use

```sh
podman compose logs -f
podman compose down
```

Update manually:

```sh
git pull --ff-only --recurse-submodules
podman compose up -d --build
```

CI tests changes; it does not deploy. To return to a version, check out
its commit and run the same build command. No custom installer, cron or
host service scripts. After reboot, run `podman compose up -d`.

## Edit

- Pages: `site/<page>/index.md` (Markdown with indented code blocks).
- Appearance: `pub/style/style.css`.
- Navigation, headers and footer: `site/_werc/lib/`.

Rebuild after editing. `./check` validates the running site (requires curl).

## Optional public tunnel

Run `cp .env.example .env` and fill in `TUNNEL_TOKEN`.
Set the Cloudflare hostname route to `http://site:8080`, then:

```sh
podman compose --profile tunnel up -d --build
```

Keep `.env` private. Set `PORT` there to change the local port.
Use `--profile tunnel` with `down` to stop both services.
For a different port, check with `./check http://localhost:8081`.

More details: [server guide](site/notes/server/index.md).

## License

Site: MIT. Oxanium: SIL OFL 1.1. werc: public domain.
