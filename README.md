# Pocket Shell extensions

Extensions for [Pocket Shell](https://github.com/satrujit11/pocket-shell): each one teaches the app how to show and control a tool on your server (containers, process managers, web servers, firewalls, databases) without any code in the app. An extension is a declarative manifest in the open `pocketshell.ext/v0` format. The Pocket Shell agent on the server runs the commands; the phone only shows the result.

| | Extension | What it does |
|---|---|---|
| <img src="docker/logo.svg" width="20" height="20" alt=""> | [Docker (manifest)](docker) | Containers of the Docker engine: state, logs, start, stop, restart, remove. |
| <img src="kubernetes/logo.svg" width="20" height="20" alt=""> | [Kubernetes](kubernetes) | Pods and nodes through kubectl: status, live logs, delete a pod. |
| <img src="nginx/logo.svg" width="20" height="20" alt=""> | [Nginx](nginx) | Web server status, enabled sites, config test, logs, reload and restart. |
| <img src="pm2/logo.svg" width="20" height="20" alt=""> | [PM2 (manifest)](pm2) | Node processes managed by PM2: status, memory, restarts, and control. |
| <img src="podman/logo.svg" width="20" height="20" alt=""> | [Podman (manifest)](podman) | Containers of Podman: state, logs, start, stop, restart, remove. |
| <img src="redis/logo.svg" width="20" height="20" alt=""> | [Redis](redis) | Server status, memory, clients and keys of a local Redis. |
|  | [Supervisor](supervisor) | Programs managed by supervisord: state, start, stop, restart. |
|  | [Services](systemd) | systemd services: what runs, what failed, logs, start, stop and restart. |
|  | [Firewall (ufw)](ufw) | Firewall status and rules: add, delete, enable and disable. |

Each folder is one extension and doubles as its wiki page:

```
<tool>/
  manifest.yaml   the extension (pocketshell.ext/v0)
  README.md       what it shows, the exact commands it runs, its buttons
  logo.svg        optional; without one the app draws the manifest's icon
  tests/          fixtures: fake programs printing real output, and what must come out
index.json        the list the app browses (checksums of every manifest)
```

## Documentation

- [Manifest standard](docs/manifest-spec.md): every field of `pocketshell.ext/v0`
- [Building an extension](docs/building.md): get the tool, write one step by step, try it on a real tool
- [Testing](docs/testing.md): fixtures with fake programs
- [Publishing and signing](docs/publishing.md): the index, signing, what the app verifies
- [Contributing](CONTRIBUTING.md): rules and the pull request checklist

## Using them

In the app: open a server, **Extensions**, **+**, then pick one from the registry. Before anything is installed you see every command it can run. The app reads `index.json` from this repository; a private extension can instead be pasted in from the **Paste** tab or kept on the server.

## Checking an extension

With the `pocket-shell-cli` tool (`ext` commands):

```sh
pocket-shell-cli ext test .                       # validate every manifest and run every fixture
pocket-shell-cli ext try redis main               # show a screen using the real tool on this machine
pocket-shell-cli ext index . > index.json         # rebuild the index after any change
```

## Adding an extension

1. Create `<tool>/manifest.yaml` with `docs:` (the tool's documentation) and, if you have one, `logo:` (a link to an SVG or PNG; without it the app uses `icon:`, or its default icon).
2. Add `tests/*.yaml`: fake programs that print what the real tool prints, and checks on screens and buttons.
3. Run `ext test`, and `ext try` against the real tool if you have it.
4. Add a `README.md` (what it shows, what it runs).
5. Rebuild `index.json` and open a pull request.

The standard is in [docs/manifest-spec.md](docs/manifest-spec.md); the full walkthrough is [docs/building.md](docs/building.md); the rules are in [CONTRIBUTING.md](CONTRIBUTING.md).

## Ids

`docker`, `podman` and `pm2` belong to the extensions built into the agent, so the manifest-only versions here are `docker-lite`, `podman-lite` and `pm2-lite`.

## Logos

Logos are from [Simple Icons](https://simpleicons.org) (CC0). The logos and names belong to their respective owners and are used only to identify each tool.

## License

Apache-2.0, see [LICENSE](LICENSE).
