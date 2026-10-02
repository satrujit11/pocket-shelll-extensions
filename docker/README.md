# <img src="logo.svg" alt="" width="48" align="left"> Docker

Containers, images, volumes and networks of Docker: state, resource use, logs, a shell, start, stop, restart, pause and remove.

`docker` v2.0.0 · [Tool documentation](https://docs.docker.com/reference/cli/docker/)

## Overview

| | |
|---|---|
| Appears when | `docker` is found on the server |
| Permissions | `read-containers`, `control-containers`, `open-shell` |
| Runs as root | no |
| Changes things | yes, through buttons you press |

## What you see

- **main** - Docker (tabs: Containers, Images, Volumes, Networks)
- **container** (tabs: Overview, Logs)
- **image**
- **volume**
- **network**

## Buttons

| Button | Safety | Notes |
|---|---|---|
| **Start** | normal | - |
| **Stop** | dangerous | asks first |
| **Restart** | normal | asks first |
| **Pause** | normal | - |
| **Unpause** | normal | - |
| **Open a shell** | normal | - |
| **Remove** | dangerous | asks first, you type the name |
| **Remove** | dangerous | asks first |
| **Remove** | dangerous | asks first |
| **Remove** | dangerous | asks first |
| **Remove unused images** | dangerous | asks first, you type the name |

## Commands it can run

Listed last because they are the fine print: the agent on the server runs all of them, never the phone, and anything you type is passed as a separate argument, never through a shell. The app shows this same list before you add the extension.

**Reads** (change nothing)

| Where | Command | As |
|---|---|---|
| `data.containers` | `docker ps -a --no-trunc --format json` | user |
| `data.images` | `docker images --format json` | user |
| `data.volumes` | `docker volume ls --format json` | user |
| `data.networks` | `docker network ls --format json` | user |
| `data.container` | `docker inspect {{ params.id }}` | user |
| `data.stats` | `docker stats --no-stream --format json {{ params.id }}` | user |
| `data.env` | `docker inspect {{ params.id }}` | user |
| `data.logs` | `docker logs --tail {{ state.lines }} --follow {{ params.id }}` (live) | user |
| `data.image` | `docker image inspect {{ params.id }}` | user |
| `data.volume` | `docker volume inspect {{ params.name }}` | user |
| `data.network` | `docker network inspect {{ params.id }}` | user |

**Changes**

| Where | Command | As |
|---|---|---|
| `actions.start` | `docker start {{ params.id }}` | user |
| `actions.stop` | `docker stop {{ params.id }}` | user |
| `actions.restart` | `docker restart {{ params.id }}` | user |
| `actions.pause` | `docker pause {{ params.id }}` | user |
| `actions.unpause` | `docker unpause {{ params.id }}` | user |
| `actions.shell` | `(script)` | user |
| `actions.remove` | `docker rm {{ params.id }}` | user |
| `actions.remove-image` | `docker rmi {{ params.id }}` | user |
| `actions.remove-volume` | `docker volume rm {{ params.name }}` | user |
| `actions.remove-network` | `docker network rm {{ params.id }}` | user |
| `actions.prune-images` | `docker image prune -f` | user |

## Test it

```sh
pocket-shell-cli ext test docker          # fake programs, what CI runs
pocket-shell-cli ext try docker main      # the real tool on this machine
```

Fixtures are in [`tests/`](tests).
