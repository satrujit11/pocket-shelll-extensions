# Kubernetes

Pods and nodes through kubectl: status, live logs, delete a pod.

| | |
|---|---|
| Extension id | `kubernetes` |
| Version | 1.0.0 |
| Shows up when | `kubectl` is installed |
| Tool documentation | https://kubernetes.io/docs/reference/kubectl/ |
| Logo | ![logo](logo.svg) |
| Permissions | read-cluster, control-cluster |

## What you see

- **main** screen: Kubernetes - tabs: Overview, Pods, Nodes
- **pod** screen: {{ params.name }} - tabs: Status, Logs

## Commands it runs

Everything below is run by the agent on the server, never by the phone. Values you enter are passed as separate arguments, never through a shell.

| Where | Kind | Command | As |
|---|---|---|---|
| `data.pods` | read | `kubectl get pods -A -o json` | user |
| `data.nodes` | read | `kubectl get nodes -o json` | user |
| `data.pod` | read | `kubectl get pod {{ params.name }} -n {{ params.ns }} -o json` | user |
| `data.logs` | read (live stream) | `kubectl logs {{ params.name }} -n {{ params.ns }} --tail 200 -f` | user |
| `actions.delete-pod` | action | `kubectl delete pod {{ params.name }} -n {{ params.ns }}` | user |

## Buttons

- **Delete pod** (`delete-pod`) - dangerous, asks first, you type a name

## Test it

```sh
# against fake programs (what CI runs)
pocket-shell-cli ext test extensions/kubernetes

# against the real tool on this machine
pocket-shell-cli ext try extensions/kubernetes main
```

The fixtures are in [`tests/`](tests).
