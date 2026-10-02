# Example extension

Copy this folder to start your own.

```sh
cp -r examples/my-timers my-extension           # change `id` and the program it looks for
pocket-shell-cli ext validate my-extension/manifest.yaml
pocket-shell-cli ext test my-extension          # runs tests/*.yaml against fake programs
pocket-shell-cli ext try my-extension main      # runs the screen with the real tools on this machine
```

Then host the folder (see [docs/hosting.md](../../docs/hosting.md)) and add its link
in the app under **Add extension > Link**.

| file | what it is |
|---|---|
| `manifest.yaml` | the extension: what to look for, what to read, what to show, which buttons |
| `tests/timers.yaml` | a fixture: fake programs and checks, so you know it works before anyone installs it |

Everything in `manifest.yaml` is explained in [docs/manifest-spec.md](../../docs/manifest-spec.md).
