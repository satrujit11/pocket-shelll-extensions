# <img src="logo.svg" alt="" width="48" align="left"> Podman (manifest)

Containers of Podman: state, logs, start, stop, restart, remove.

`podman-lite` v1.0.0 · [Tool documentation](https://docs.podman.io/en/latest/Commands.html)

## Overview

| | |
|---|---|
| Appears when | `podman` is found on the server |
| Permissions | `read-containers`, `control-containers` |
| Runs as root | no |
| Changes things | yes, through buttons you press |

## What you see

- **main** - Containers (tabs: Overview, Running, All)
- **detail** (tabs: Status, Logs)

## Buttons

| Button | Safety | Notes |
|---|---|---|
| **Start** | normal | - |
| **Stop** | dangerous | asks first |
| **Restart** | normal | asks first |
| **Remove** | dangerous | asks first, you type the name |

## Commands it can run

Listed last because they are the fine print: the agent on the server runs all of them, never the phone, and anything you type is passed as a separate argument, never through a shell. The app shows this same list before you add the extension.

**Reads** (change nothing)

| Where | Command | As |
|---|---|---|
| `data.containers` | `podman ps -a --no-trunc --format json` | user |
| `data.container` | `podman inspect {{ params.id }}` | user |
| `data.logs` | `podman logs --tail 200 --follow {{ params.id }}` (live) | user |

**Changes**

| Where | Command | As |
|---|---|---|
| `actions.start` | `podman start {{ params.id }}` | user |
| `actions.stop` | `podman stop {{ params.id }}` | user |
| `actions.restart` | `podman restart {{ params.id }}` | user |
| `actions.remove` | `podman rm {{ params.id }}` | user |

## Test it

```sh
pocket-shell-cli ext test podman          # fake programs, what CI runs
pocket-shell-cli ext try podman main      # the real tool on this machine
```

Fixtures are in [`tests/`](tests).
