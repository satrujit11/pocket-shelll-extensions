# Building and trying an extension

You need the `pocket-shell-cli` tool. It contains the same engine the agent on a server uses, so what passes here behaves the same on a phone.

## Get the tool

```sh
git clone https://github.com/satrujit11/pocket-shell-cli
cd pocket-shell-cli
go build -o pocket-shell-cli .      # Go 1.25 or newer
```

Put the binary on your `PATH`, or call it by its path. Everything below uses `pocket-shell-cli`.

Its `ext` commands:

| command | what it does |
|---|---|
| `ext validate FILE...` | checks manifests against the standard and says where each problem is |
| `ext test DIR...` | validates `DIR/manifest.yaml`, then runs every `DIR/tests/*.yaml` against fake programs; `ext test .` does every extension folder of a catalog |
| `ext try DIR SCREEN [k=v...]` | resolves a screen with the **real** programs on this machine and prints the JSON the phone would get |
| `ext try DIR SCREEN --action NAME [--form k=v] [--confirm] [--confirm-text TEXT]` | presses a button for real (it still enforces confirmation, forms and conditions) |
| `ext index DIR [--key FILE]` | writes `index.json` for a catalog (checksums, logos, docs; signed when `--key` is given) |
| `ext keygen FILE`, `ext sign FILE --key KEY` | make a signing key; sign one manifest |

## Write the extension, step by step

1. **Find the commands.** Run the tool's own commands that print machine-readable output (`--format json`, `-o json`, `jlist`). JSON is the easiest. For text output use the `table`, `keyvalue`, `regex` or `lines` parsers (see the spec's data sources section).
2. **Create the folder:** `mytool/manifest.yaml`. A minimal one:

   ```yaml
   spec: pocketshell.ext/v0
   id: mytool
   name: My tool
   description: "What it shows, in one line."
   icon: server
   category: system
   version: 1.0.0
   author: You
   docs: https://example.org/mytool/docs
   detect:
     binaries: [mytool]
   data:
     items:
       run: [mytool, list, --json]
       parse: json
       path: $
       refresh: 10s
   screens:
     main:
       title: My tool
       subtitle: "{{ data.items | count() }} items"
       sections:
         - components:
             - type: list
               from: items
               search: true
               item:
                 title: "{{ row.name }}"
                 subtitle: "{{ row.status }}"
   ```

3. **Add a button** under `actions:` and list it in the screen's `actions:` (or on the detail screen):

   ```yaml
   actions:
     restart:
       label: Restart
       icon: restart
       run: [mytool, restart, "{{ params.name }}"]
       confirm: { title: "Restart {{ params.name }}?", text: It is unavailable for a moment. }
   ```

4. **Add a detail screen** with `params:` and make list rows open it with `on_tap: { screen: detail, params: { name: "{{ row.name }}" } }`. A source that needs a parameter lists it in `params:` and uses `{{ params.name }}`.
5. **Live logs:** a source with `parse: lines`, `stream: true` (and `merge_stderr: true` if the tool prints logs on stderr) shown by a `logs` component follows the command while the screen is open.
6. **Expressions** live inside `{{ }}`: `row.name`, `data.items | where(row.status == 'up') | count()`, `row.size | bytes()`. Use parentheses before a pipe after arithmetic: `(row.used * 1024) | bytes()`. Full list in the spec.
7. **Check the files**: `pocket-shell-cli ext validate mytool/manifest.yaml`.

## Try it on a real tool

```sh
pocket-shell-cli ext try mytool main                       # the main screen, real commands
pocket-shell-cli ext try mytool detail name=web            # a screen with a parameter
pocket-shell-cli ext try mytool detail name=web --action restart --confirm
```

This is the quickest way to find a wrong path or parser: the output is the exact JSON the app draws, and a data source that failed appears under `errors`.

## Test it with fakes

See [testing.md](testing.md). `pocket-shell-cli ext test mytool` is what must pass before a pull request.

## See it on a phone

From the app: open a server, **Extensions**, **+**, **Paste**, paste your `manifest.yaml`, **Review** (you see every command and script it would run), **Add to this server**, then **Install** and open it. Nothing leaves your server and nothing is published.

## Logos and docs

`docs:` is required for this catalog. For a logo, add `mytool/logo.svg` and set `logo:` to its raw GitHub URL:
`https://raw.githubusercontent.com/satrujit11/pocket-shelll-extensions/main/mytool/logo.svg`. The app draws SVGs in a single tint (so single-colour marks work best) and falls back to `icon:` if the logo cannot be loaded.
