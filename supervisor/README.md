# Supervisor

Programs managed by supervisord: state, start, stop, restart.

| | |
|---|---|
| Extension id | `supervisor` |
| Version | 1.0.0 |
| Shows up when | `supervisorctl` is installed |
| Tool documentation | http://supervisord.org/running.html#supervisorctl-command-line-options |
| Logo | none (the app draws the `process` icon) |
| Permissions | read-processes, control-processes, sudo |

## What you see

- **main** screen: Supervisor - tabs: Overview, Programs
- **detail** screen: {{ params.name }} - tabs: Status, Output

## Commands it runs

Everything below is run by the agent on the server, never by the phone. Values you enter are passed as separate arguments, never through a shell.

| Where | Kind | Command | As |
|---|---|---|---|
| `data.programs` | read | `supervisorctl status` | sudo |
| `data.program` | read | `supervisorctl status {{ params.name }}` | sudo |
| `data.logs` | read | `supervisorctl tail {{ params.name }}` | sudo |
| `actions.start` | action | `supervisorctl start {{ params.name }}` | sudo |
| `actions.stop` | action | `supervisorctl stop {{ params.name }}` | sudo |
| `actions.restart` | action | `supervisorctl restart {{ params.name }}` | sudo |

## Buttons

- **Start** (`start`)
- **Stop** (`stop`) - dangerous, asks first
- **Restart** (`restart`) - asks first

## Test it

```sh
# against fake programs (what CI runs)
pocket-shell-cli ext test extensions/supervisor

# against the real tool on this machine
pocket-shell-cli ext try extensions/supervisor main
```

The fixtures are in [`tests/`](tests).
