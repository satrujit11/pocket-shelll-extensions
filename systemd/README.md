# Services

systemd services: what runs, what failed, logs, start, stop and restart.

`systemd` v1.2.0 · [Tool documentation](https://www.freedesktop.org/software/systemd/man/latest/systemctl.html)

## Overview

| | |
|---|---|
| Appears when | `systemctl` is found on the server |
| Permissions | `read-services`, `control-services`, `sudo` |
| Runs as root | some commands (sudo) |
| Changes things | yes, through buttons you press |

## What you see

- **main** - Services
- **detail** (tabs: Status, Logs)

## Buttons

| Button | Safety | Notes |
|---|---|---|
| **Start** | normal | - |
| **Stop** | dangerous | asks first, you type the name |
| **Restart** | normal | asks first |
| **Reload systemd** | normal | - |

## Commands it can run

Listed last because they are the fine print: the agent on the server runs all of them, never the phone, and anything you type is passed as a separate argument, never through a shell. The app shows this same list before you add the extension.

**Reads** (change nothing)

| Where | Command | As |
|---|---|---|
| `data.units` | `systemctl list-units --type=service --all --no-pager --output=json` | user |
| `data.unit` | `systemctl show {{ params.name }} --no-pager` | user |
| `data.journal` | `journalctl -u {{ params.name }} -n {{ state.lines }} -f --no-pager --output=short-iso` (live) | sudo |

**Changes**

| Where | Command | As |
|---|---|---|
| `actions.start` | `systemctl start {{ params.name }}` | sudo |
| `actions.stop` | `systemctl stop {{ params.name }}` | sudo |
| `actions.restart` | `systemctl restart {{ params.name }}` | sudo |
| `actions.reload-daemon` | `systemctl daemon-reload` | sudo |

## Test it

```sh
pocket-shell-cli ext test systemd          # fake programs, what CI runs
pocket-shell-cli ext try systemd main      # the real tool on this machine
```

Fixtures are in [`tests/`](tests).
