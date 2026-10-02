# Publishing, index and signing

## The index

`index.json` lists every extension (id, name, description, icon, logo, docs, version, author, the path of its manifest and its SHA-256). The app reads it from the default branch:

```
https://raw.githubusercontent.com/satrujit11/pocket-shelll-extensions/main/index.json
```

Rebuild it after every change to a manifest (a stale checksum makes the app refuse that manifest):

```sh
pocket-shell-cli ext index . > index.json
```

## Signing (maintainers)

Signing lets the app label an entry **Verified**. How the keys work, how to sign, and how to run your own registry: [signing.md](signing.md). In short, from the Pocket Shell app repository:

```sh
tool/sign_registry.sh ../pocket-shelll-extensions ../pocket-shell-cli
```

## What the app and agent check

When someone adds an extension from the registry:

1. The app downloads `manifest.yaml` and checks it against the index's SHA-256.
2. The server's agent previews it: it validates it, rejects an id the server already lists, and lists every command and script. The person sees all of it before choosing **Add to this server**.
3. The agent checks the checksum and, when the app pinned a key and the entry is signed, the signature, and only then writes the file.
4. The extension is stored as a file on that server (`~/.pocket_shell/extensions/`) and listed in that server's database. Editing the file later shows a "Modified" badge.

## Moving or mirroring the catalog

Nothing in a manifest refers to where the catalog is hosted except `logo:` links (raw URLs of the logo files). To host it elsewhere, copy this repository and set `defaultRegistryUrl` in the app (or build the app with `--dart-define=EXT_REGISTRY_URL=<your index.json URL>`), then rebuild the index.

## Releasing

1. Merge pull requests that pass `ext test`.
2. Rebuild and sign `index.json` (see signing.md) in the same commit as the manifest changes.
3. Bump an extension's `version` when its behaviour changes.
