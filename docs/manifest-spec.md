# Pocket Shell extension standard, v0

An extension is **one file** (YAML or JSON) that tells Pocket Shell how to find a
tool on a server, what data to read from it, how to lay that data out, and what
buttons act on it. It contains **no code**. The agent on the server runs the
commands and evaluates the expressions; the app only draws what the agent
resolves. That is what lets any app that speaks the standard show any extension.

    spec: pocketshell.ext/v0

## 1. Identity

| field | meaning |
|---|---|
| `spec` | must be `pocketshell.ext/v0` |
| `id` | lowercase `a-z0-9-`, 1-40 chars, unique per server |
| `name`, `description` | shown in the extensions list |
| `icon` | a name from the icon set (section 9) |
| `logo` | optional: an `http`/`https` link to an SVG or PNG of the tool. The app draws it (SVGs in one tint, like the other icons); without a logo, or if it cannot be loaded, it draws `icon`, or its default icon |
| `category` | free text group: `containers`, `processes`, `web`, `database`, `system`, ... |
| `version` | semver of the extension |
| `author`, `homepage`, `license` | optional |
| `requires` | components the extension needs, e.g. `[chart, wizard]`; an app that lacks one says "needs a newer app" |

## Links to documentation

`docs:` (also `homepage:`) is an `http` or `https` link. It may be set on the
extension, on a screen (`screens.<name>.docs`, falling back to the extension's)
and on an action (`actions.<name>.docs`). The app shows it as a book icon on the
extension, in the add dialog, on the screen's bar and in an action's
confirmation. Private extensions may point at internal wikis. The agent never
fetches a link; the app opens it in the browser and refuses any other scheme.

## 2. Detection

How the agent decides the tool is on this server.

    detect:
      binaries: [docker]          # any of these on PATH
      paths: [/etc/nginx]         # or any of these paths exists
      version_args: [--version]   # run with the found binary; first line is shown
      services: [nginx]           # systemd unit exists (optional)

At least one of `binaries`, `paths`, `services`. Detection never changes anything.

## 3. Data sources

Named, read-only reads that views bind to.

    data:
      services:
        run: [systemctl, list-units, --type=service, --all, --output=json]
        parse: json                 # json | jsonl | lines | table | keyvalue | regex | text
        path: $                     # where the rows are inside the parsed value
        refresh: 10s                # 2s..1h, or `manual`, or `open` (once per screen open)
        timeout: 5s
        privilege: user             # user | sudo
      unit:                          # parametrised: used by a detail screen
        params: [name]
        run: [systemctl, show, "{{ params.name }}"]
        parse: keyvalue

`run` is an **argument list**, never a shell string, so values cannot inject
commands. `{{ }}` is only allowed inside an argument and is quoted as one
argument. Parsers:

* `json`: the output is JSON; `path` selects rows (JSONPath subset).
* `jsonl`: one JSON value per line (what `docker ps --format json` prints); one row each.
* `lines`: one row per line, `{line}`.
* `table`: whitespace or fixed columns: `columns: [name, status, ...]`, `skip: 1`.
* `keyvalue`: `Key=Value`, `Key: Value` or `key:value` lines into one object.
* `regex`: `pattern` with named groups, one row per match.
* `text`: the whole output as `{text}`.

**Live streams.** `stream: true` (with `parse: lines`, and no `refresh`) keeps the
command running while a screen that shows it is open and publishes its lines as
they arrive (live logs, e.g. `journalctl -f`). The agent keeps the last 1000
lines, runs the command once however many phones watch, and stops it about 25
seconds after the last phone looks away. A `logs` component binds to it like any
other source. Form fields of type `path` let the person browse the server's
folders (`file.list`).

**Inline scripts.** Instead of `run`, a data source or an action may carry a
`script`, run by an `interpreter` (`sh` by default; `bash`, `python3`, `node`)
with `args` as its arguments:

    data:
      summary:
        script: |
          read l1 _ < /proc/loadavg
          printf '{"load":%s}\n' "$l1"
        parse: json
    actions:
      check:
        interpreter: bash
        script: 'ping -c 3 -W 2 "$1"'
        args: ["{{ form.host }}"]

A script is fixed text of at most 16 KiB. Values from screens and forms reach it
only through `args` (`$1`, `$2`... in sh and bash, `sys.argv[1:]` in python3), so
they can never become part of the code. `{{ }}` is not evaluated inside a
script. `privilege: sudo` runs the interpreter itself through `sudo -n`. Before
installing, the app shows every script in full next to the commands.

Limits: output is truncated at 1 MiB; a source that exceeds its `timeout` yields an error state shown in the view, never a hang.

## 4. Expressions

`{{ ... }}` in strings. Evaluated by the agent in a sandbox with no side
effects: field access (`row.name`), comparison, `&&`, `||`, `!`, `in`, ternary,
string and list functions (`count()`, `sum()`, `where()`, `map()`, `join()`,
`upper()`, `lower()`, `contains()`, `startsWith()`, `bytes()`, `duration()`,
`ago()`, `percent()`). No loops, no assignment, no network, no files. The
language is a deliberate subset of CEL.

Available names: `data.<source>` (the rows), `row` (inside a list or table),
`params` (screen parameters), `selection`, `form` (an action's fields), `server`
(`name`, `user`).

## 5. Screens, tabs, sections

    screens:
      main:                          # `main` is the entry
        title: "{{ server.name }}: Services"
        subtitle: "{{ data.services | count() }} units"
        actions: [reload-all]        # buttons in the app bar
        tabs:                        # optional; without tabs use `sections`
          - id: running
            title: Running
            badge: "{{ data.services | where(row.sub == 'running') | count() }}"
            sections: [...]
      detail:
        params: [name]               # screens can take parameters
        sections: [...]

A **section** groups components:

    - title: Status
      layout: { columns: 2 }         # 1 | 2 | 3; collapses to 1 on a phone
      collapsible: true
      visible: "{{ data.unit.ActiveState != '' }}"
      components: [...]

## 6. Components

Every component has `type`, optional `visible`, `title`, and `span` (columns it spans).

| type | shows | key fields |
|---|---|---|
| `stat` | a big value with a label | `value`, `label`, `detail`, `tone`, `unit`, `trend` |
| `stats` | a row/grid of `stat` | `items: [stat]` |
| `card` | a titled box that holds components, may be tappable | `components`, `on_tap` |
| `list` | rows from a source | `from`, `where` (keep rows for which it holds), `item: {title, subtitle, leading, trailing, badge, tone}`, `on_tap`, `empty`, `search`, `group_by`, `sort` |
| `table` | columns from a source | `from`, `where`, `columns: [{title, value, align, width}]`, `on_tap`, `sort`, `paging` |
| `keyvalue` | label/value pairs | `items: [{label, value, copy, tone}]` |
| `chart` | line, area, bar, gauge, sparkline | `kind`, `series: [{from, x, y, label, tone}]`, `window`, `unit` |
| `progress` | a bar | `value`, `max`, `label`, `tone` |
| `badge` / `chips` | status pills | `text`, `tone`, `icon` |
| `logs` | streaming or paged text | `from`, `follow`, `search`, `levels` |
| `text` | markdown (no HTML) | `value` |
| `code` | monospace block, copyable | `value`, `language` |
| `banner` | a notice | `text`, `tone`, `actions` |
| `buttons` | a row of actions | `items: [action id]` |
| `empty` | a placeholder | `title`, `text`, `icon`, `actions` |
| `divider`, `spacer` | spacing | |

`tone` is semantic only: `neutral`, `success`, `warning`, `danger`, `info`,
`muted`. Extensions cannot set colours, fonts, HTML or pixel sizes: the app owns
the look, so every extension matches the app, light or dark.

Navigation: `on_tap: { screen: detail, params: { name: "{{ row.unit }}" } }`, or
`on_tap: { action: restart, with: { name: "{{ row.unit }}" } }`, or
`on_tap: { url: ... }` (https only, opens outside the app).

## 7. Actions (buttons, forms, steps)

    actions:
      restart:
        label: Restart
        icon: restart
        style: default               # default | primary | danger
        visible: "{{ row.active }}"
        enabled: "{{ !row.masked }}"
        run: [systemctl, restart, "{{ params.name }}"]
        privilege: sudo
        confirm:
          title: "Restart {{ params.name }}?"
          text: This briefly interrupts the service.
          type_to_confirm: "{{ params.name }}"   # optional: type the name to proceed
        after: [refresh: services]   # sources to re-read when it succeeds
        result: toast                # toast | output | none
      create-site:
        label: New site
        steps:                       # a wizard: several pages
          - title: Domain
            fields:
              - { id: domain, type: text, label: Domain, required: true, pattern: "^[a-z0-9.-]+$" }
          - title: Options
            fields:
              - { id: proxy, type: select, label: Upstream, options: "{{ data.upstreams | map(row.name) }}" }
              - { id: tls, type: toggle, label: Enable HTTPS, default: true }
          - title: Review
            review: true              # shows the answers before running
        run: [create-site, "{{ form.domain }}", "{{ form.proxy }}"]

Field types: `text`, `number`, `password`, `textarea`, `select`, `multiselect`,
`toggle`, `path` (server file picker), `duration`. Validation (`required`,
`pattern`, `min`, `max`, `min_length`) runs in the app and again in the agent.
`stream: true` on an action shows its output live. A `danger` action always
asks for confirmation, whatever the manifest says.

## 8. Permissions and safety

    permissions: [read-services, control-services, sudo]

Declared up front and shown at install: what it reads, whether it changes
anything, whether it uses sudo. Rules the agent enforces for every extension:

* commands are argument lists; values are never joined into a shell string
* it can only run commands its manifest declares, with the declared arguments
* `privilege: sudo` needs the permission `sudo` and the person's consent
* output limits, timeouts and rate limits (a `refresh` below 2s is refused)
* a manifest that fails validation is listed as broken and never runs
* private extensions live on the server; a public one is pinned by hash

## 9. Icons

A fixed set of names the app maps to its own icons (`docker`, `podman`, `pm2`,
`server`, `database`, `web`, `cloud`, `lock`, `clock`, `cpu`, `memory`, `disk`,
`network`, `play`, `stop`, `restart`, `trash`, `edit`, `add`, `search`, `file`,
`folder`, `terminal`, `warning`, `check`, `info`, `settings`, `shield`,
`certificate`, `container`, `process`, `schedule`).

## 10. Built-in entries

Docker, Podman and PM2 are extensions too. Until their screens are rewritten in
this vocabulary they use `entry: { type: builtin, target: docker }`, which opens
the app's own native screen. Only extensions shipped with the agent may use
`builtin`.

## 11. Compatibility

`spec` is versioned. A v0 manifest keeps working in later apps; new fields are
additive. Unknown fields are rejected at validation (so a typo is caught), and
an unknown component type in a screen is shown as "not supported by this app"
instead of breaking the screen.
