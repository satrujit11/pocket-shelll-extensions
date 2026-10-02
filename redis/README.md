# <img src="logo.svg" alt="" width="48" align="left"> Redis

Server status, memory, clients and keys of a local Redis.

`redis` v1.0.0 · [Tool documentation](https://redis.io/docs/latest/commands/info/)

## Overview

| | |
|---|---|
| Appears when | `redis-cli` is found on the server |
| Permissions | `read-redis`, `control-redis` |
| Runs as root | no |
| Changes things | yes, through buttons you press |

## What you see

- **main** - Redis (tabs: Overview)

## Buttons

| Button | Safety | Notes |
|---|---|---|
| **Save to disk** | normal | asks first |

## Commands it can run

Listed last because they are the fine print: the agent on the server runs all of them, never the phone, and anything you type is passed as a separate argument, never through a shell. The app shows this same list before you add the extension.

**Reads** (change nothing)

| Where | Command | As |
|---|---|---|
| `data.info` | `redis-cli info` | user |

**Changes**

| Where | Command | As |
|---|---|---|
| `actions.save` | `redis-cli bgsave` | user |

## Test it

```sh
pocket-shell-cli ext test redis          # fake programs, what CI runs
pocket-shell-cli ext try redis main      # the real tool on this machine
```

Fixtures are in [`tests/`](tests).
