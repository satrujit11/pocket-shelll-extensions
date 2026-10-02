# PM2 (manifest)

Node processes managed by PM2: status, memory, restarts, and control.

| | |
|---|---|
| Extension id | `pm2-lite` |
| Version | 1.0.0 |
| Shows up when | `pm2` is installed |
| Tool documentation | https://pm2.keymetrics.io/docs/usage/quick-start/ |
| Logo | ![logo](logo.svg) |
| Permissions | read-processes, control-processes |

## What you see

- **main** screen: Processes - tabs: Overview, All
- **detail** screen: {{ params.name }} - tabs: Status, Logs

## Commands it runs

Everything below is run by the agent on the server, never by the phone. Values you enter are passed as separate arguments, never through a shell.

| Where | Kind | Command | As |
|---|---|---|---|
| `data.logs` | read (live stream) | `pm2 logs {{ params.name }} --lines 100 --raw` | user |
| `data.procs` | read | `pm2 jlist` | user |
| `actions.restart` | action | `pm2 restart {{ params.name }}` | user |
| `actions.stop` | action | `pm2 stop {{ params.name }}` | user |
| `actions.start` | action | `pm2 start {{ params.name }}` | user |
| `actions.delete` | action | `pm2 delete {{ params.name }}` | user |

## Buttons

- **Restart** (`restart`) - asks first
- **Stop** (`stop`) - dangerous, asks first
- **Start** (`start`)
- **Delete** (`delete`) - dangerous, asks first, you type a name

## Test it

```sh
# against fake programs (what CI runs)
pocket-shell-cli ext test extensions/pm2

# against the real tool on this machine
pocket-shell-cli ext try extensions/pm2 main
```

The fixtures are in [`tests/`](tests).
