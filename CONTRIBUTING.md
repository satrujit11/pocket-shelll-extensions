# Contributing

Thank you for adding to the catalog. An extension here is a manifest (no code in the app), so review is mostly about three things: it is **honest** about what it runs, it is **safe** to run on someone's server, and it is **tested**.

## The short version

1. Fork this repository and create a folder `<tool>/` (lowercase, the tool's name).
2. Write `<tool>/manifest.yaml` following [docs/manifest-spec.md](docs/manifest-spec.md). Start from a similar extension in this repository.
3. Write `<tool>/tests/*.yaml` ([docs/testing.md](docs/testing.md)).
4. Run `pocket-shell-cli ext test <tool>` until it passes; if the tool is on your machine, also `pocket-shell-cli ext try <tool> main`. ([docs/building.md](docs/building.md) shows how to get the tool.)
5. Write `<tool>/README.md` (copy one and adjust: what it shows, the exact commands, the buttons).
6. Rebuild the index: `pocket-shell-cli ext index . > index.json`.
7. Open a pull request.

## What a manifest must have

- `spec: pocketshell.ext/v0`, a unique `id`, `name`, `description`, `version`, `author`.
- `docs:` a link to the tool's own documentation (or to your wiki page for it).
- `logo:` (optional) a link to an SVG or PNG of the tool. Without it the app draws `icon:`, or its default icon. Put the file in `<tool>/logo.svg` and link it with its raw URL. Only submit logos you may redistribute (Simple Icons logos are CC0; the marks stay their owners').
- `detect:` how to tell the tool is installed (`binaries`, `paths` or `services`). Detection must never change anything.
- `permissions:` honest names for what it reads and changes, including `sudo` if any command uses `privilege: sudo`.

## Rules for commands

- `run` is an argument list, never a shell string. Values from screens and forms go in as their own list items.
- Reads (`data:`) must not change anything. Everything that changes the server is an `actions:` button.
- A button that is destructive (stops, deletes, restarts, overwrites) has `style: danger` or a `confirm:`; deleting something that cannot be brought back asks the person to type its name (`type_to_confirm`).
- Use `privilege: sudo` only where the tool needs it and declare the `sudo` permission. The agent runs `sudo -n`, so nothing ever asks for a password.
- Inline `script:` is allowed (see the spec) but keep it short and readable: people read every script before they add the extension. Pass values with `args:` and read them as `$1`, `$2`; never build commands from text.
- No network access to anything the tool does not itself need, no downloading and running code, no credentials in the manifest.

## Tests

Every extension needs at least one fixture. A fixture names fake programs that print what the real tool prints (copy real output), then checks the screens and buttons. It must also prove the safety rules: a dangerous button refuses without confirmation, and the right command is what runs. See [docs/testing.md](docs/testing.md).

## Ids

`docker`, `podman` and `pm2` belong to the extensions built into the agent. A manifest-only version uses another id (`docker-lite`).

## Style

- Titles and labels are short and in plain words. Say what a button does.
- Prefer a few good tabs over many sections; put the most useful thing first.
- Don't copy a tool's whole interface. Show what someone checking a server from a phone needs, and the few actions they would take.

## Reviewing

A maintainer reads the manifest the way a person would before installing it (the app shows every command), runs `ext test`, and merges. The index is rebuilt from the manifests, and signed by a maintainer with the registry key ([docs/publishing.md](docs/publishing.md)). Unsigned entries still install, labelled "from a registry, not signed".
