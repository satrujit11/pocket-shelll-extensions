# <img src="logo.svg" alt="" width="48" align="left"> PM2 (manifest)

Node processes managed by PM2: status, memory, restarts, and control.

`pm2-lite` v1.0.0 · [Tool documentation](https://pm2.keymetrics.io/docs/usage/quick-start/)

## Overview

| | |
|---|---|
| Appears when | `pm2` is found on the server |
| Permissions | `read-processes`, `control-processes` |
| Runs as root | no |
| Changes things | yes, through buttons you press |

## What you see

- **main** - Processes (tabs: Overview, All)
- **detail** (tabs: Status, Logs)

## Buttons

| Button | Safety | Notes |
|---|---|---|
| **Restart** | normal | asks first |
| **Stop** | dangerous | asks first |
| **Start** | normal | - |
| **Delete** | dangerous | asks first, you type the name |

## Commands it can run

Listed last because they are the fine print: the agent on the server runs all of them, never the phone, and anything you type is passed as a separate argument, never through a shell. The app shows this same list before you add the extension.

**Reads** (change nothing)

| Where | Command | As |
|---|---|---|
| `data.logs` | `pm2 logs {{ params.name }} --lines 100 --raw` (live) | user |
| `data.procs` | `pm2 jlist` | user |

**Changes**

| Where | Command | As |
|---|---|---|
| `actions.restart` | `pm2 restart {{ params.name }}` | user |
| `actions.stop` | `pm2 stop {{ params.name }}` | user |
| `actions.start` | `pm2 start {{ params.name }}` | user |
| `actions.delete` | `pm2 delete {{ params.name }}` | user |

## Test it

```sh
pocket-shell-cli ext test pm2          # fake programs, what CI runs
pocket-shell-cli ext try pm2 main      # the real tool on this machine
```

Fixtures are in [`tests/`](tests).
