# Nginx

Web server status, enabled sites, config test, logs, reload and restart.

| | |
|---|---|
| Extension id | `nginx` |
| Version | 1.0.0 |
| Shows up when | `nginx` is installed |
| Tool documentation | https://nginx.org/en/docs/ |
| Logo | ![logo](logo.svg) |
| Permissions | read-config, read-logs, control-services, sudo |

## What you see

- **main** screen: Nginx - tabs: Overview, Sites, Logs
- **site** screen: {{ params.name }}

## Commands it runs

Everything below is run by the agent on the server, never by the phone. Values you enter are passed as separate arguments, never through a shell.

| Where | Kind | Command | As |
|---|---|---|---|
| `data.status` | read | `systemctl is-active nginx` | user |
| `data.configtest` | read | `nginx -t` | sudo |
| `data.sites` | read | `ls -1 /etc/nginx/sites-enabled` | user |
| `data.site` | read | `cat /etc/nginx/sites-enabled/{{ params.name }}` | sudo |
| `data.errors` | read | `tail -n 300 /var/log/nginx/error.log` | sudo |
| `data.access` | read | `tail -n 300 /var/log/nginx/access.log` | sudo |
| `actions.test` | action | `nginx -t` | sudo |
| `actions.reload` | action | `systemctl reload nginx` | sudo |
| `actions.restart` | action | `systemctl restart nginx` | sudo |

## Buttons

- **Test config** (`test`)
- **Reload** (`reload`) - asks first
- **Restart** (`restart`) - dangerous, asks first

## Test it

```sh
# against fake programs (what CI runs)
pocket-shell-cli ext test extensions/nginx

# against the real tool on this machine
pocket-shell-cli ext try extensions/nginx main
```

The fixtures are in [`tests/`](tests).
