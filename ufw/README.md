# Firewall (ufw)

Firewall status and rules: add, delete, enable and disable.

`ufw` v1.0.0 · [Tool documentation](https://manpages.ubuntu.com/manpages/noble/en/man8/ufw.8.html)

## Overview

| | |
|---|---|
| Appears when | `ufw` is found on the server |
| Permissions | `read-firewall`, `control-firewall`, `sudo` |
| Runs as root | some commands (sudo) |
| Changes things | yes, through buttons you press |

## What you see

- **main** - Firewall (tabs: Status, Rules)
- **rule**

## Buttons

| Button | Safety | Notes |
|---|---|---|
| **Allow a port** | normal | asks for details |
| **Block a port** | normal | asks first; asks for details |
| **Delete rule** | dangerous | asks first |
| **Enable** | normal | asks first |
| **Disable** | dangerous | asks first, you type the name |

## Commands it can run

Listed last because they are the fine print: the agent on the server runs all of them, never the phone, and anything you type is passed as a separate argument, never through a shell. The app shows this same list before you add the extension.

**Reads** (change nothing)

| Where | Command | As |
|---|---|---|
| `data.status` | `ufw status verbose` | sudo |
| `data.rules` | `ufw status numbered` | sudo |

**Changes**

| Where | Command | As |
|---|---|---|
| `actions.allow` | `ufw allow {{ form.port }}/{{ form.proto }}` | sudo |
| `actions.deny` | `ufw deny {{ form.port }}/{{ form.proto }}` | sudo |
| `actions.delete-rule` | `ufw --force delete {{ params.n }}` | sudo |
| `actions.enable` | `ufw --force enable` | sudo |
| `actions.disable` | `ufw disable` | sudo |

## Test it

```sh
pocket-shell-cli ext test ufw          # fake programs, what CI runs
pocket-shell-cli ext try ufw main      # the real tool on this machine
```

Fixtures are in [`tests/`](tests).
