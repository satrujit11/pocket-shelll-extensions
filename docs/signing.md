# Signing extensions

A signature answers one question for the app: *is this manifest exactly what the
registry's maintainer published?* A signed entry is labelled **Verified**; an
unsigned one is labelled "From a registry, not signed" and can still be added
after the person has read every command it runs. A signature never replaces that
review: it says who published the file, not that the file is harmless.

## How it works

* The registry has one **ed25519 key pair**. The **public** key is built into the
  app (`registryPublicKey`); the **private** key signs.
* `index.json` has, for every extension, the SHA-256 of its `manifest.yaml` and
  a signature over exactly the manifest's bytes.
* When someone adds an extension, the app checks the SHA-256, the server's agent
  checks the SHA-256 again and verifies the signature with the pinned public key,
  and only then writes the file. A changed byte, a different key or a missing
  signature all end as "not signed" (or, if the checksum is wrong, a refusal).

The public key of this registry:

    key id      d77edb48cdea
    public key  uiqTGupHsEXAT+uVPy0HI9bQ50kmUxjzxnW4MuUs0CI=

## Sign (the maintainer)

The private key is kept in the Pocket Shell app's private repository, in
`signing/registry.key`, with a script that does everything below.

    # in the app repository
    tool/sign_registry.sh ../pocket-shelll-extensions ../pocket-shell-cli
    cd ../pocket-shelll-extensions
    git add -A && git commit -m "Update extensions" && git push

The script builds the CLI, runs every extension's fixtures, rebuilds
`index.json` with the checksums and signatures, and writes it here.

**Sign last.** The signature covers the manifest byte for byte, so any later edit
(even whitespace, even by a formatter) breaks it. Commit the manifest and the
index together; a contributor's pull request does not touch `index.json`, the
maintainer re-signs after merging.

By hand, with the CLI:

    pocket-shell-cli ext keygen registry.key                      # once; prints the public key
    pocket-shell-cli ext index . --key registry.key > index.json  # every manifest
    pocket-shell-cli ext sign --key registry.key docker/manifest.yaml   # one file: prints its sha256 and signature

## Check a signature

    pocket-shell-cli ext test docker              # fixtures
    POCKET_SHELL_REGISTRY_PUBKEY='uiqTGupHsEXAT+uVPy0HI9bQ50kmUxjzxnW4MuUs0CI=' \
    POCKET_SHELL_EXTENSIONS=. go test ./internal/extensions -run Catalog   # in pocket-shell-cli

The second command checks that every checksum is current, every signature
verifies with the key above, and that the copies of Docker, Podman and PM2 that
ship with the agent equal the files here.

## Run your own registry

1. `pocket-shell-cli ext keygen my.key`: keep `my.key` out of git and offline.
2. Lay out a folder per extension (`<id>/manifest.yaml`, `tests/`, optional `README.md`, `logo.svg`).
3. `pocket-shell-cli ext index . --key my.key > index.json`, publish the folder over HTTPS.
4. Build the app with your index URL and public key: set `defaultRegistryUrl`
   (or `--dart-define=EXT_REGISTRY_URL=https://.../index.json`) and
   `registryPublicKey` in `lib/modules/agent/extension_registry.dart`.

## If the key leaks

Make a new key (`ext keygen`), put its public half in the app, publish a new app
version and re-sign the index. Until people update, an app with the old key
trusts what the old key signed; the old key must therefore be treated as
compromised the moment it leaks.

## Private extensions

An extension a person pastes in, or writes into `~/.pocket_shell/extensions/`,
is never signed and is shown as private. It is allowed to do exactly what its
manifest says, and the person sees all of it before adding it.
