# Services

systemd services: what runs, what failed, logs, start, stop and restart.

| | |
|---|---|
| Extension id | `systemd` |
| Version | 1.0.0 |
| Shows up when | `systemctl` is installed |
| Tool documentation | https://www.freedesktop.org/software/systemd/man/latest/systemctl.html |
| Logo | none (the app draws the `server` icon) |
| Permissions | read-services, control-services, sudo |

## What you see

- **main** screen: Services - tabs: Overview, Running, All
- **detail** screen: {{ params.name }} - tabs: Status, Logs

## Commands it runs

Everything below is run by the agent on the server, never by the phone. Values you enter are passed as separate arguments, never through a shell.

| Where | Kind | Command | As |
|---|---|---|---|
| `data.units` | read | `systemctl list-units --type=service --all --no-pager --output=json` | user |
| `data.unit` | read | `systemctl show {{ params.name }} --no-pager` | user |
| `data.journal` | read (live stream) | `journalctl -u {{ params.name }} -n 200 -f --no-pager --output=short-iso` | sudo |
| `actions.start` | action | `systemctl start {{ params.name }}` | sudo |
| `actions.stop` | action | `systemctl stop {{ params.name }}` | sudo |
| `actions.restart` | action | `systemctl restart {{ params.name }}` | sudo |
| `actions.reload-daemon` | action | `systemctl daemon-reload` | sudo |

## Buttons

- **Start** (`start`)
- **Stop** (`stop`) - dangerous, asks first, you type a name
- **Restart** (`restart`) - asks first
- **Reload systemd** (`reload-daemon`)

## Test it

```sh
# against fake programs (what CI runs)
pocket-shell-cli ext test extensions/systemd

# against the real tool on this machine
pocket-shell-cli ext try extensions/systemd main
```

The fixtures are in [`tests/`](tests).
