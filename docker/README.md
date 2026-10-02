# Docker (manifest)

Containers of the Docker engine: state, logs, start, stop, restart, remove.

| | |
|---|---|
| Extension id | `docker-lite` |
| Version | 1.0.0 |
| Shows up when | `docker` is installed |
| Tool documentation | https://docs.docker.com/reference/cli/docker/ |
| Logo | ![logo](logo.svg) |
| Permissions | read-containers, control-containers |

## What you see

- **main** screen: Containers - tabs: Overview, Running, All
- **detail** screen: {{ data.container.Name | replace('/', '') }} - tabs: Status, Logs

## Commands it runs

Everything below is run by the agent on the server, never by the phone. Values you enter are passed as separate arguments, never through a shell.

| Where | Kind | Command | As |
|---|---|---|---|
| `data.containers` | read | `docker ps -a --no-trunc --format json` | user |
| `data.container` | read | `docker inspect {{ params.id }}` | user |
| `data.logs` | read (live stream) | `docker logs --tail 200 --follow {{ params.id }}` | user |
| `actions.start` | action | `docker start {{ params.id }}` | user |
| `actions.stop` | action | `docker stop {{ params.id }}` | user |
| `actions.restart` | action | `docker restart {{ params.id }}` | user |
| `actions.remove` | action | `docker rm {{ params.id }}` | user |

## Buttons

- **Start** (`start`)
- **Stop** (`stop`) - dangerous, asks first
- **Restart** (`restart`) - asks first
- **Remove** (`remove`) - dangerous, asks first, you type a name

## Test it

```sh
# against fake programs (what CI runs)
pocket-shell-cli ext test extensions/docker

# against the real tool on this machine
pocket-shell-cli ext try extensions/docker main
```

The fixtures are in [`tests/`](tests).
