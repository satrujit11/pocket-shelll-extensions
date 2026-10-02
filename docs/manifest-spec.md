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
| `errors` | optional: hints for known failures (section 10) |
| `logo` | optional: an `http`/`https` link to an SVG or PNG of the tool. The app draws it (SVGs in one tint, like the other icons); without a logo, or if it cannot be loaded, it draws `icon`, or its default icon |
| `category` | free text group: `containers`, `processes`, `web`, `database`, `system`, ... |
| `version` | semver of the extension |
| `author`, `homepage`, `license` | optional |
| `requires` | features the extension needs, e.g. `[properties, tree, controls, streams, filters, row-actions, sheet, terminal, state, history, chart, wizard]`; an app that lacks one says "needs a newer app" |

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

**Conditional sources (`when`).** A source with `when: "{{ state.env == true }}"`
is read only while the condition holds. Use it for anything the person should
ask for first (environment variables, secrets): until then the command is never
run and its data never leaves the server.

**Dependencies.** A source whose `run` or `args` mention `data.<other>` waits for
that source and uses its rows. This is how a log screen finds the file to tail:

    data:
      procs: { run: [pm2, jlist], parse: json, path: $, refresh: 5s }
      out:
        params: [name]
        run: [tail, -n, "{{ state.lines }}", -F, "{{ (data.procs | where(row.name == params.name) | first()).pm2_env.pm_out_log_path }}"]
        parse: lines
        stream: true

Sources with no dependency run in parallel; the rest follow in waves. A cycle is
rejected at validation. If a source fails, the sources that depend on it are
skipped and the screen says why.

**History.** `history: 30` keeps the last 30 values the agent read (one per
`refresh`). `data.<name>` is still the latest read; `history.<name>` is a list of
`{t, v}` for charts and sparklines.

## 4. Expressions

`{{ ... }}` in strings. Evaluated by the agent in a sandbox with no side
effects: field access (`row.name`), comparison, `&&`, `||`, `!`, `in`, ternary,
string and list functions (`count()`, `sum()`, `where()`, `map()`, `join()`,
`upper()`, `lower()`, `contains()`, `startsWith()`, `bytes()`, `duration()`,
`ago()`, `percent()`). No loops, no assignment, no network, no files. The
language is a deliberate subset of CEL.

Available names: `data.<source>` (the rows), `history.<source>`, `row` (inside a
list, table or `each` group), `params` (screen parameters), `state` (the screen's
own controls, section 5), `form` (an action's fields), `server` (`name`, `user`).

Also `first()`, `last()`, `take(n)`, `reverse()`, `unique()`, `sort()`, `keys()`,
`default(x)`, `trim()`, `replace(a, b)`, `split(s)`, `round()`, `floor()`,
`ceil()`, `number()`, `string()`. `ago()` of a "never" time (year 1) is empty.

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

**Screen state and controls.** A screen can hold values the person changes (how
many log lines, whether to reveal secrets). Declare them and refer to them as
`state.<name>`; the app sends them back with the screen request, so the agent
re-resolves it:

    screens:
      container:
        state:
          lines: { type: number, label: Lines, options: [100, 200, 500, 1000], default: 200 }
          env:   { type: toggle, label: Show environment variables, auth: true, default: false }

Types: `select`, `toggle`, `text`, `number`. A `select` or `number` with `options` is drawn as one compact drop-down row (label left, choice right, options in the app's sheet); `toggle` as a switch; `text` as a field. A state variable with `auth: true`
asks for the app's PIN or biometrics before it leaves its default. Put a control
next to what it changes with `controls: [lines]` on any component, or draw
several together with a `controls` component (`items: [env]`). `state.<name>` may
be used in `run`, `when`, `visible`, titles and every template.

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
| `list` | rows from a source | `from`, `where`, `item: {title, subtitle, leading, trailing, badge, tone, actions}`, `on_tap`, `empty`, `search`, `filters`, `group_by`, `sort` |
| `table` | columns from a source | `from`, `where`, `columns: [{title, value, align, width}]`, `on_tap`, `sort`, `paging` |
| `keyvalue` | label/value pairs | `items: [{label, value, copy, tone}]` |
| `chart` | line, area, bar, gauge, sparkline | `kind`, `series: [{from, x, y, label, tone}]`, `window`, `unit` |
| `progress` | a bar | `value`, `max`, `label`, `tone` |
| `badge` / `chips` | status pills | `text`, `tone`, `icon` |
| `logs` | live or paged text | `from` or `streams`, `follow`, `search`, `levels`, `wrap`, `controls` |
| `properties` | an object shown in classified groups | `value`, `groups`, `rest`, `exclude`, `empty` |
| `tree` | a raw object, keys open and close | `value`, `depth` |
| `controls` | the screen's own state controls | `items: [state names]` |
| `text` | markdown (no HTML) | `value` |
| `code` | monospace block, copyable | `value`, `language` |
| `banner` | a notice | `text`, `tone`, `actions` |
| `buttons` | a row of actions | `items: [action id]` |
| `empty` | a placeholder | `title`, `text`, `icon`, `actions` |
| `divider`, `spacer` | spacing | |

`tone` is semantic only: `neutral`, `success`, `warning`, `danger`, `info`,
`muted`. Extensions cannot set colours, fonts, HTML or pixel sizes: the app owns
the look, so every extension matches the app, light or dark.

### Lists: search, chips, row buttons

    - type: list
      from: containers
      search: true                       # a search field above the list, outside its card
      filters:                           # chips above the list; "All" is added
        - { id: running, label: Running, where: "{{ row.State == 'running' }}" }
        - { id: stopped, label: Stopped, where: "{{ row.State != 'running' }}" }
      item:
        title: "{{ row.Names }}"
        subtitle: "{{ row.Image }}"
        actions:                         # buttons at the end of each row
          - { action: remove-image, with: { id: "{{ row.ID }}" } }
      on_tap: { screen: container, params: { id: "{{ row.ID }}" } }

A row with one action shows it as an icon button, several fold into a menu. A
row shows a chevron only when it has `on_tap`. Rows act on themselves through
`with`: the action reads them as `params`.

### Properties: show everything, classified

`properties` is how a manifest shows the data of one object (a `docker inspect`)
so that nothing is lost and the important parts come first.

    - type: properties
      value: "{{ data.container }}"
      groups:
        - title: Configuration
          fields:
            - { label: Image, path: Config.Image, copy: true }
            - { label: ID, path: Id, copy: true, mono: true }
            - { label: Restart policy, path: HostConfig.RestartPolicy.Name, hide_empty: true }
        - title: Status
          fields:
            - { label: Started, path: State.StartedAt, format: ago }
            - { label: Error, path: State.Error, hide_empty: true, tone: danger }
        - title: "{{ row.Destination }}"          # one group per element
          each: "{{ data.container.Mounts }}"
          fields:
            - { label: Source, value: "{{ row.Source }}", copy: true }
      rest: tree                                  # everything the groups did not claim
      rest_title: Everything else
      exclude: [Config.Env]                       # keep this out of the tree

A field takes its value from `path` (dotted, `Mounts.0.Source`) or `value` (an
expression). `format`: `bytes`, `duration`, `ago`, `date`, `percent`, `yesno`,
`list`, `json`, `upper`, `lower`. Also `copy`, `secret` (hidden until the person
taps the eye), `mono`, `tone`, `link`, `hide_empty`, `empty` (what to print when
there is no value), and `visible`. A group may be `collapsible`, `hide_empty`
(drop it when it has no fields) or have an `empty` text.

`rest: tree` is the guarantee that a manifest author never has to list every key
of a large document: the classified part reads well, and the raw rest stays one
tap away. Keep secrets out of the rest with `exclude`.

### Logs

    - type: logs
      follow: true
      search: true                        # filter box above the log, outside it
      wrap: false                         # the person can toggle it
      controls: [lines]                   # a state control shown with it
      streams:                            # one log, or several to switch between
        - { id: out, label: Output, from: out }
        - { id: err, label: Errors, from: err, tone: danger }
      levels:                             # colour a line by the first rule it matches
        - { match: "error|fatal|panic", level: error }    # error | warn | info | debug | trace
        - { match: "warn", level: warn }

### Navigation

`on_tap: { screen: detail, params: { name: "{{ row.unit }}" } }` opens a screen. Add
`as: sheet` to open it in a bottom sheet over the current one (good for a look at
an image or a volume), `as: page` (default) for a full page. Or
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

**Terminal actions.** `terminal: [docker, exec, -it, "{{ params.id }}", /bin/sh]`
instead of `run` opens the app's terminal on this server and types the command
into it. The agent runs nothing for it; the person sees and controls the shell.
The command is an argument list, quoted for the shell; it is listed with the
other commands before install.

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
`network`, `play`, `stop`, `pause`, `restart`, `trash`, `edit`, `add`, `search`, `file`,
`folder`, `terminal`, `warning`, `check`, `info`, `settings`, `shield`,
`certificate`, `container`, `process`, `schedule`).

## 10. Errors and what they mean

Every source and button can fail, and the person should read why in plain words
rather than a raw message. The agent matches the error text against `errors`
(first match wins; any case; at the extension, the source or the action) and the
app shows the hint above the raw message:

    errors:
      - match: "permission denied"
        title: Docker refuses this user
        text: Add the user to the docker group, or open this server as root.
        docs: https://docs.docker.com/engine/install/linux-postinstall/
      - match: "Cannot connect to the Docker daemon"
        title: Docker is not running
        text: Start the service and open this screen again.

Defaults exist for the usual ones (a command that is not found, a timeout,
`sudo` that needs a password, permission denied). A failed source never blanks
the screen: the components that do not need it still show, and the screen
carries a note per failure. An action that fails shows its output.

## 11. Designing screens (guidance, applied by the app)

The app owns the look; a manifest owns the structure. These rules make a
manifest read well on a phone and are what the catalog follows.

1. **One page for one subject.** Do not split what belongs together into tabs
   (a firewall's status and rules is one page). Use tabs for different kinds of
   thing (containers / images / volumes / networks) or for a heavy view such as
   logs. At most four tabs; more become scrolling underline tabs.
2. **Search and filters sit outside the card they filter.** `search: true` and
   `filters` draw above the list.
3. **A row you can tap looks tappable and one you cannot does not.** The chevron
   comes only from `on_tap`. Do not open a detail page just to show a delete
   button: put the button on the row with `item.actions`.
4. **Put the answer first.** A row of `stat` components, then the list, then the
   fine print in a `collapsible` section.
5. **Classify, do not dump.** Show an object as `properties` groups and keep the
   rest in `rest: tree`. Group names say what the person is looking for
   (Configuration, Status, Network, Mounts), not what the tool calls it.
6. **Secrets are asked for.** Put them behind a `when` source and an `auth`
   toggle, and keep them out of `rest` with `exclude`.
7. **Say what a button does and when.** Use `visible` so only the buttons that
   fit the state appear; `confirm` for anything that interrupts; `type_to_confirm`
   for anything that deletes.
8. **Several logs are streams of one component**, not separate pages.
9. **Show an empty state**, with the way out (`empty`, or an `empty` component
   with a button), and let errors explain themselves with `errors`.
10. **Link the documentation** (`docs`) on the extension, its screens and its
    dangerous actions.

## 12. Compatibility

`spec` is versioned. A v0 manifest keeps working in later apps; new fields are
additive. Unknown fields are rejected at validation (so a typo is caught), and
an unknown component type in a screen is shown as "not supported by this app"
instead of breaking the screen.
