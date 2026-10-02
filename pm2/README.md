# <img src="logo.svg" alt="" width="48" align="left"> PM2

Node processes managed by PM2: status, resource use, restarts, error and output logs, start, stop, restart, reload and delete.

`pm2` v2.0.0 · [Tool documentation](https://pm2.keymetrics.io/docs/usage/quick-start/)

## Overview

| | |
|---|---|
| Appears when | `pm2` is found on the server |
| Permissions | `read-processes`, `control-processes` |
| Runs as root | no |
| Changes things | yes, through buttons you press |

## What you see

- **main** - Processes
- **detail** (tabs: Overview, Logs)

## Buttons

| Button | Safety | Notes |
|---|---|---|
| **Start** | normal | - |
| **Restart** | normal | asks first |
| **Reload (no downtime)** | normal | - |
| **Stop** | dangerous | asks first |
| **Clear its logs** | normal | asks first |
| **Delete** | dangerous | asks first, you type the name |
| **Save the list** | normal | - |

## Commands it can run

Listed last because they are the fine print: the agent on the server runs all of them, never the phone, and anything you type is passed as a separate argument, never through a shell. The app shows this same list before you add the extension.

**Reads** (change nothing)

| Where | Command | As |
|---|---|---|
| `data.procs` | `pm2 jlist` | user |
| `data.envvars` | `pm2 jlist` | user |
| `data.out` | `tail -n {{ state.lines }} -F {{ (data.procs | where(row.name == params.name) | first()).pm2_env.pm_out_log_path }}` (live) | user |
| `data.err` | `tail -n {{ state.lines }} -F {{ (data.procs | where(row.name == params.name) | first()).pm2_env.pm_err_log_path }}` (live) | user |

**Changes**

| Where | Command | As |
|---|---|---|
| `actions.start` | `pm2 start {{ params.name }}` | user |
| `actions.restart` | `pm2 restart {{ params.name }}` | user |
| `actions.reload` | `pm2 reload {{ params.name }}` | user |
| `actions.stop` | `pm2 stop {{ params.name }}` | user |
| `actions.flush` | `pm2 flush {{ params.name }}` | user |
| `actions.delete` | `pm2 delete {{ params.name }}` | user |
| `actions.save` | `pm2 save` | user |

## Test it

```sh
pocket-shell-cli ext test pm2          # fake programs, what CI runs
pocket-shell-cli ext try pm2 main      # the real tool on this machine
```

Fixtures are in [`tests/`](tests).
