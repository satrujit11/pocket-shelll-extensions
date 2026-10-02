# Firewall (ufw)

Firewall status and rules: add, delete, enable and disable.

| | |
|---|---|
| Extension id | `ufw` |
| Version | 1.0.0 |
| Shows up when | `ufw` is installed |
| Tool documentation | https://manpages.ubuntu.com/manpages/noble/en/man8/ufw.8.html |
| Logo | none (the app draws the `shield` icon) |
| Permissions | read-firewall, control-firewall, sudo |

## What you see

- **main** screen: Firewall - tabs: Status, Rules
- **rule** screen: Rule {{ params.n }}

## Commands it runs

Everything below is run by the agent on the server, never by the phone. Values you enter are passed as separate arguments, never through a shell.

| Where | Kind | Command | As |
|---|---|---|---|
| `data.status` | read | `ufw status verbose` | sudo |
| `data.rules` | read | `ufw status numbered` | sudo |
| `actions.allow` | action | `ufw allow {{ form.port }}/{{ form.proto }}` | sudo |
| `actions.deny` | action | `ufw deny {{ form.port }}/{{ form.proto }}` | sudo |
| `actions.delete-rule` | action | `ufw --force delete {{ params.n }}` | sudo |
| `actions.enable` | action | `ufw --force enable` | sudo |
| `actions.disable` | action | `ufw disable` | sudo |

## Buttons

- **Allow a port** (`allow`) - asks for details
- **Block a port** (`deny`) - asks first, asks for details
- **Delete rule** (`delete-rule`) - dangerous, asks first
- **Enable** (`enable`) - asks first
- **Disable** (`disable`) - dangerous, asks first, you type a name

## Test it

```sh
# against fake programs (what CI runs)
pocket-shell-cli ext test extensions/ufw

# against the real tool on this machine
pocket-shell-cli ext try extensions/ufw main
```

The fixtures are in [`tests/`](tests).
