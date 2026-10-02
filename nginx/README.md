# <img src="logo.svg" alt="" width="48" align="left"> Nginx

Web server status, enabled sites, config test, logs, reload and restart.

`nginx` v1.2.0 · [Tool documentation](https://nginx.org/en/docs/)

## Overview

| | |
|---|---|
| Appears when | `nginx` is found on the server |
| Permissions | `read-config`, `read-logs`, `control-services`, `sudo` |
| Runs as root | some commands (sudo) |
| Changes things | yes, through buttons you press |

## What you see

- **main** - Nginx (tabs: Overview, Logs)
- **site**

## Buttons

| Button | Safety | Notes |
|---|---|---|
| **Test config** | normal | - |
| **Reload** | normal | asks first |
| **Restart** | dangerous | asks first |

## Commands it can run

Listed last because they are the fine print: the agent on the server runs all of them, never the phone, and anything you type is passed as a separate argument, never through a shell. The app shows this same list before you add the extension.

**Reads** (change nothing)

| Where | Command | As |
|---|---|---|
| `data.status` | `systemctl is-active nginx` | user |
| `data.configtest` | `nginx -t` | sudo |
| `data.sites` | `ls -1 /etc/nginx/sites-enabled` | user |
| `data.site` | `cat /etc/nginx/sites-enabled/{{ params.name }}` | sudo |
| `data.errors` | `tail -n {{ state.lines }} -F /var/log/nginx/error.log` (live) | sudo |
| `data.access` | `tail -n {{ state.lines }} -F /var/log/nginx/access.log` (live) | sudo |

**Changes**

| Where | Command | As |
|---|---|---|
| `actions.test` | `nginx -t` | sudo |
| `actions.reload` | `systemctl reload nginx` | sudo |
| `actions.restart` | `systemctl restart nginx` | sudo |

## Test it

```sh
pocket-shell-cli ext test nginx          # fake programs, what CI runs
pocket-shell-cli ext try nginx main      # the real tool on this machine
```

Fixtures are in [`tests/`](tests).
