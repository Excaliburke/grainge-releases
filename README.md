# Grainge — releases

Downloads for [Grainge](https://github.com/Excaliburke/grainworks), and the
feed its update check reads.

This repo is separate from the source, and public, for one reason: a release
asset on a private repo needs a token to download, and a token shipped inside
a client that anyone can open is not a secret. The same split is used by
`flyover-releases` and `houndstooth-releases`.

## What the app reads

Grainge checks for updates only when someone presses **CHECK FOR UPDATES** in
its Help panel — there is no check on launch and no timer. What it reads is
this repo's newest release, through the GitHub Releases API:

```
https://api.github.com/repos/Excaliburke/grainge-releases/releases/latest
```

Not the `releases/latest/download/…` asset path that the other two apps use.
That path redirects twice onto `release-assets.githubusercontent.com` and no
hop in the chain sends an `access-control-allow-origin` header, so a web view
cannot read it — and all three Grainge hosts (the single-file app, the macOS
app's WKWebView, and the Photoshop UXP panel) are web views. `api.github.com`
answers `access-control-allow-origin: *`.

From that response it reads:

| field       | used for                                              |
|-------------|-------------------------------------------------------|
| `tag_name`  | the version. One leading `v` is stripped              |
| `body`      | the release notes, collapsed to one line and clipped  |
| `html_url`  | where a person is told to go                          |

So **the tag is the version**. Tag a release `v1.0.1` and a copy running 1.0.0
will offer it; tag it anything that is not three plain numbers after the `v`
and the app says so rather than guessing, because `1.0.10` sorts below
`1.0.9` as a string and that is an update that otherwise silently never
arrives.

## Cutting a release

1. `./mac/release.sh --notarize grainary` in the source repo — builds, signs,
   notarizes, staples, and writes `mac/build/Grainge-<version>.dmg`.
2. Bump the version in all three places that state it: `js/version.js`,
   `manifest.json`, and `CFBundleShortVersionString` in `mac/build.sh`.
   `node dev/updates.test.mjs` fails if they disagree.
3. Create the release here with the tag `v<version>` and attach the DMG.
