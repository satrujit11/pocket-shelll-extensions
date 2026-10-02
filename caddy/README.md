# <img src="logo.svg" alt="" width="48" align="left"> Caddy

Web server status, the sites in the Caddyfile, config validation, live logs, reload and restart.

`caddy` v1.0.0 · [Tool documentation](https://caddyserver.com/docs/)

## Overview

| | |
|---|---|
| Appears when | `caddy` is found on the server |
| Permissions | `read-config`, `read-logs`, `control-services`, `sudo` |
| Runs as root | some commands (sudo) |
| Changes things | yes, through buttons you press |

## What you see

- **main** - Caddy (tabs: Overview, Logs)
- **site**

## Buttons

| Button | Safety | Notes |
|---|---|---|
| **Validate config** | normal | - |
| **Reload** | normal | asks first |
| **Restart** | dangerous | asks first |
| **Format the Caddyfile** | dangerous | asks first, you type the name |

## Commands it can run

Listed last because they are the fine print: the agent on the server runs all of them, never the phone, and anything you type is passed as a separate argument, never through a shell. The app shows this same list before you add the extension.

**Reads** (change nothing)

| Where | Command | As |
|---|---|---|
| `data.status` | `systemctl is-active caddy` | user |
| `data.validate` | `caddy validate --config /etc/caddy/Caddyfile` | sudo |
| `data.caddyfile` | `cat /etc/caddy/Caddyfile` | sudo |
| `data.sites` | `(script)` (sh script) | sudo |
| `data.site` | `(script) {{ params.name }}` (sh script) | sudo |
| `data.journal` | `journalctl -u caddy -n {{ state.lines }} -f --no-pager --output=short-iso` (live) | sudo |

**Changes**

| Where | Command | As |
|---|---|---|
| `actions.validate` | `caddy validate --config /etc/caddy/Caddyfile` | sudo |
| `actions.reload` | `systemctl reload caddy` | sudo |
| `actions.restart` | `systemctl restart caddy` | sudo |
| `actions.format` | `caddy fmt --overwrite /etc/caddy/Caddyfile` | sudo |

<details><summary>Script of <code>data.sites</code> (sh)</summary>

Arguments reach it as `$1`, `$2`... and are never part of its text.

```sh
# The address of every top-level block of the Caddyfile.
awk '/^[^[:space:]#}(][^{]*\{[[:space:]]*$/ { sub(/[[:space:]]*\{.*/, ""); print }' /etc/caddy/Caddyfile
```
</details>

<details><summary>Script of <code>data.site</code> (sh)</summary>

Arguments reach it as `$1`, `$2`... and are never part of its text.

```sh
# The block whose address is $1, up to its closing brace.
awk -v want="$1" '
  !on && /^[^[:space:]#}(][^{]*\{[[:space:]]*$/ { a=$0; sub(/[[:space:]]*\{.*/, "", a); if (a == want) { on=1 } }
  on { print }
  on && /^\}/ { exit }
' /etc/caddy/Caddyfile
```
</details>

## Test it

```sh
pocket-shell-cli ext test caddy          # fake programs, what CI runs
pocket-shell-cli ext try caddy main      # the real tool on this machine
```

Fixtures are in [`tests/`](tests).
