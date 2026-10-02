# <img src="logo.svg" alt="" width="48" align="left"> Podman

Containers, images, volumes and networks of Podman: state, resource use, logs, a shell, start, stop, restart, pause and remove.

`podman` v2.0.0 · [Tool documentation](https://docs.podman.io/en/latest/Commands.html)

## Overview

| | |
|---|---|
| Appears when | `podman` is found on the server |
| Permissions | `read-containers`, `control-containers`, `open-shell` |
| Runs as root | no |
| Changes things | yes, through buttons you press |

## What you see

- **main** - Podman (tabs: Containers, Images, Volumes, Networks)
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
| `data.containers` | `podman ps -a --no-trunc --format json` | user |
| `data.images` | `podman images --format json` | user |
| `data.volumes` | `podman volume ls --format json` | user |
| `data.networks` | `podman network ls --format json` | user |
| `data.container` | `podman inspect {{ params.id }}` | user |
| `data.stats` | `podman stats --no-stream --format json {{ params.id }}` | user |
| `data.env` | `podman inspect {{ params.id }}` | user |
| `data.logs` | `podman logs --tail {{ state.lines }} --follow {{ params.id }}` (live) | user |
| `data.image` | `podman image inspect {{ params.id }}` | user |
| `data.volume` | `podman volume inspect {{ params.name }}` | user |
| `data.network` | `podman network inspect {{ params.id }}` | user |

**Changes**

| Where | Command | As |
|---|---|---|
| `actions.start` | `podman start {{ params.id }}` | user |
| `actions.stop` | `podman stop {{ params.id }}` | user |
| `actions.restart` | `podman restart {{ params.id }}` | user |
| `actions.pause` | `podman pause {{ params.id }}` | user |
| `actions.unpause` | `podman unpause {{ params.id }}` | user |
| `actions.shell` | `(script)` | user |
| `actions.remove` | `podman rm {{ params.id }}` | user |
| `actions.remove-image` | `podman rmi {{ params.id }}` | user |
| `actions.remove-volume` | `podman volume rm {{ params.name }}` | user |
| `actions.remove-network` | `podman network rm {{ params.id }}` | user |
| `actions.prune-images` | `podman image prune -f` | user |

## Test it

```sh
pocket-shell-cli ext test podman          # fake programs, what CI runs
pocket-shell-cli ext try podman main      # the real tool on this machine
```

Fixtures are in [`tests/`](tests).
