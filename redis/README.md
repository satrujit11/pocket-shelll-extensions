# Redis

Server status, memory, clients and keys of a local Redis.

| | |
|---|---|
| Extension id | `redis` |
| Version | 1.0.0 |
| Shows up when | `redis-cli` is installed |
| Tool documentation | https://redis.io/docs/latest/commands/info/ |
| Logo | ![logo](logo.svg) |
| Permissions | read-redis, control-redis |

## What you see

- **main** screen: Redis - tabs: Overview

## Commands it runs

Everything below is run by the agent on the server, never by the phone. Values you enter are passed as separate arguments, never through a shell.

| Where | Kind | Command | As |
|---|---|---|---|
| `data.info` | read | `redis-cli info` | user |
| `actions.save` | action | `redis-cli bgsave` | user |

## Buttons

- **Save to disk** (`save`) - asks first

## Test it

```sh
# against fake programs (what CI runs)
pocket-shell-cli ext test extensions/redis

# against the real tool on this machine
pocket-shell-cli ext try extensions/redis main
```

The fixtures are in [`tests/`](tests).
