# <img src="logo.svg" alt="" width="48" align="left"> Kubernetes

Pods and nodes through kubectl: status, live logs, delete a pod.

`kubernetes` v1.1.0 · [Tool documentation](https://kubernetes.io/docs/reference/kubectl/)

## Overview

| | |
|---|---|
| Appears when | `kubectl` is found on the server |
| Permissions | `read-cluster`, `control-cluster` |
| Runs as root | no |
| Changes things | yes, through buttons you press |

## What you see

- **main** - Kubernetes (tabs: Pods, Nodes)
- **pod** (tabs: Status, Logs)

## Buttons

| Button | Safety | Notes |
|---|---|---|
| **Delete pod** | dangerous | asks first, you type the name |

## Commands it can run

Listed last because they are the fine print: the agent on the server runs all of them, never the phone, and anything you type is passed as a separate argument, never through a shell. The app shows this same list before you add the extension.

**Reads** (change nothing)

| Where | Command | As |
|---|---|---|
| `data.pods` | `kubectl get pods -A -o json` | user |
| `data.nodes` | `kubectl get nodes -o json` | user |
| `data.pod` | `kubectl get pod {{ params.name }} -n {{ params.ns }} -o json` | user |
| `data.logs` | `kubectl logs {{ params.name }} -n {{ params.ns }} --tail {{ state.lines }} -f` (live) | user |

**Changes**

| Where | Command | As |
|---|---|---|
| `actions.delete-pod` | `kubectl delete pod {{ params.name }} -n {{ params.ns }}` | user |

## Test it

```sh
pocket-shell-cli ext test kubernetes          # fake programs, what CI runs
pocket-shell-cli ext try kubernetes main      # the real tool on this machine
```

Fixtures are in [`tests/`](tests).
