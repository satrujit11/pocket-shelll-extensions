# Supervisor

Programs managed by supervisord: state, start, stop, restart.

`supervisor` v1.0.0 · [Tool documentation](http://supervisord.org/running.html#supervisorctl-command-line-options)

## Overview

| | |
|---|---|
| Appears when | `supervisorctl` is found on the server |
| Permissions | `read-processes`, `control-processes`, `sudo` |
| Runs as root | some commands (sudo) |
| Changes things | yes, through buttons you press |

## What you see

- **main** - Supervisor (tabs: Overview, Programs)
- **detail** (tabs: Status, Output)

## Buttons

| Button | Safety | Notes |
|---|---|---|
| **Start** | normal | - |
| **Stop** | dangerous | asks first |
| **Restart** | normal | asks first |

## Commands it can run

Listed last because they are the fine print: the agent on the server runs all of them, never the phone, and anything you type is passed as a separate argument, never through a shell. The app shows this same list before you add the extension.

**Reads** (change nothing)

| Where | Command | As |
|---|---|---|
| `data.programs` | `supervisorctl status` | sudo |
| `data.program` | `supervisorctl status {{ params.name }}` | sudo |
| `data.logs` | `supervisorctl tail {{ params.name }}` | sudo |

**Changes**

| Where | Command | As |
|---|---|---|
| `actions.start` | `supervisorctl start {{ params.name }}` | sudo |
| `actions.stop` | `supervisorctl stop {{ params.name }}` | sudo |
| `actions.restart` | `supervisorctl restart {{ params.name }}` | sudo |

## Test it

```sh
pocket-shell-cli ext test supervisor          # fake programs, what CI runs
pocket-shell-cli ext try supervisor main      # the real tool on this machine
```

Fixtures are in [`tests/`](tests).
