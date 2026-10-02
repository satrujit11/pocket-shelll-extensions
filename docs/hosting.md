# Host your own extensions

You can add an extension to a server from any link the phone can reach, without
the public registry: in the app, **Add extension > Link**. A link can point at

* **one manifest** (`.../manifest.yaml`), or
* **a catalog**: an `index.json` with the manifests beside it, like this repository. The app lists what is in it and you pick one.

Either way the app shows every command the extension can run before anything is
installed, exactly as for the registry. A link you add is **not signed** (unless
you add your own signature, see [signing.md](signing.md)), so read what it will run.

## Build it first

1. Copy [examples/my-timers](../examples/my-timers) and change `id`, the program it detects, and the screens.
2. `pocket-shell-cli ext test my-extension` checks it against fake programs;
   `ext try my-extension main` runs a screen with the real tools on your machine
   ([building.md](building.md)).
3. For a catalog, put each extension in its own folder, then
   `pocket-shell-cli ext index . > index.json`.

```
my-extensions/
  index.json            # made by `ext index .`  (a catalog only)
  caddy/manifest.yaml
  caddy/tests/...
  mytool/manifest.yaml
```

## Where to put it

### GitHub or GitLab, public

Push the folder. In the app, paste the repository or folder address. These all work:

```
https://github.com/you/my-extensions                      # a catalog (index.json at the top)
https://github.com/you/my-extensions/tree/main/mytool     # one extension's folder
https://github.com/you/my-extensions/blob/main/mytool/manifest.yaml
https://raw.githubusercontent.com/you/my-extensions/main/index.json
```

GitHub addresses are turned into their `raw.githubusercontent.com` form for you.
Raw files are cached for about five minutes after a push.

### GitHub, private repository (with a token)

1. On GitHub: **Settings > Developer settings > Personal access tokens > Fine-grained tokens > Generate**.
2. Give it **access to only this repository** and one permission: **Contents: Read-only**. Set an expiry.
3. In the app: **Add extension > Link**, paste the address, paste the token into **Access token**, and leave **Remember for github.com** on if you want it kept.

The token is kept in the phone's secure storage and is sent **only** to the host
of the link you entered (it is never sent to the server, never written in a
manifest, and never sent to any other host, even after a redirect). **Forget the
token** from the same screen. Other hosts that take a bearer token in the
`Authorization` header (GitLab with a project or deploy token, Gitea, a private
web server) work the same way.

### Amazon S3, CloudFront or any HTTPS host

* **Public bucket or CloudFront:** upload the folder and use the HTTPS address of `index.json` or a manifest.
* **Private bucket:** S3 cannot take a bearer token, and the phone must not hold AWS credentials. Put CloudFront (or any web server) in front and protect it with a header token, or add **one manifest** with a pre-signed URL (`aws s3 presign s3://bucket/mytool/manifest.yaml --expires-in 3600`).
  A pre-signed URL cannot be used for a **catalog**, because the files beside the index would each need their own signature.
* Serve plain files over **HTTPS** (`http://` is refused, except for `localhost` while you develop). Keep each manifest under 512 KiB.

## What the phone does and does not do with your link

* It downloads the file(s) itself, then sends the text to the agent on the server, which validates it again.
* The agent never fetches a link, and no extension can make the agent or the app fetch anything.
* A catalog may only point at files on its own host.

## Updating

Publish a higher `version` in the manifest (and rebuild `index.json`). For a
catalog link, add the link again: the new version replaces the old one and keeps it switched on.
