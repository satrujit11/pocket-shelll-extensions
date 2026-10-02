# Podman (manifest)

Containers of Podman: state, logs, start, stop, restart, remove.

| | |
|---|---|
| Extension id | `podman-lite` |
| Version | 1.0.0 |
| Shows up when | `podman` is installed |
| Tool documentation | https://docs.podman.io/en/latest/Commands.html |
| Logo | ![logo](logo.svg) |
| Permissions | read-containers, control-containers |

## What you see

- **main** screen: Containers - tabs: Overview, Running, All
- **detail** screen: {{ data.container.Name }} - tabs: Status, Logs

## Commands it runs

Everything below is run by the agent on the server, never by the phone. Values you enter are passed as separate arguments, never through a shell.

| Where | Kind | Command | As |
|---|---|---|---|
| `data.containers` | read | `podman ps -a --no-trunc --format json` | user |
| `data.container` | read | `podman inspect {{ params.id }}` | user |
| `data.logs` | read (live stream) | `podman logs --tail 200 --follow {{ params.id }}` | user |
| `actions.start` | action | `podman start {{ params.id }}` | user |
| `actions.stop` | action | `podman stop {{ params.id }}` | user |
| `actions.restart` | action | `podman restart {{ params.id }}` | user |
| `actions.remove` | action | `podman rm {{ params.id }}` | user |

## Buttons

- **Start** (`start`)
- **Stop** (`stop`) - dangerous, asks first
- **Restart** (`restart`) - asks first
- **Remove** (`remove`) - dangerous, asks first, you type a name

## Test it

```sh
# against fake programs (what CI runs)
pocket-shell-cli ext test extensions/podman

# against the real tool on this machine
pocket-shell-cli ext try extensions/podman main
```

The fixtures are in [`tests/`](tests).
