# AdNauseam Lite: Chromium Manifest V3 development build

This branch contains the in-progress Manifest V3 port of AdNauseam. It produces
an unpacked Chromium extension that Chrome 139 and later can load, but it is not
a stable release and does not yet provide complete feature parity with the
Manifest V2 extension.

Use this build for development and testing. If you need the current supported
AdNauseam experience, use one of the browsers listed in the
[upstream installation documentation](https://github.com/dhowe/AdNauseam/wiki/FAQ#how-do-i-install-adnauseam).

## Why this build exists

Chrome removed Manifest V2 support after Chrome 138. Current AdNauseam Chromium
release artifacts still use Manifest V2, so developer mode alone cannot load
them in current Chrome versions.

AdNauseam Lite uses:

- a Manifest V3 service worker;
- declarative network request rulesets generated from AdNauseam, uBlock, and
  community filter lists;
- content scripts for cosmetic filtering and ad collection; and
- an offscreen document for visit operations that need a DOM context.

The Chromium MV3 manifest retains the key used by the existing unpacked
Chromium package. Chrome therefore derives the established extension ID:

```text
dkoaabhijcomjinndlgbmfnmnjnmdeeb
```

The manifest key is a public identifier, not a signing secret. Keeping it stable
lets an in-place unpacked update retain the same Chrome extension identity.
Storage migration between the MV2 and MV3 implementations should still be
treated as developmental and backed up before testing.

## Current status

The collection, storage, and visit backend is largely implemented. The build
also includes the popup, settings surface, and a compatibility-wrapped version
of the vault.

Known gaps include:

- collected image persistence can fall back to placeholders;
- visit scheduling does not yet survive every service-worker suspension;
- idle-aware visit scheduling is incomplete;
- notifications and EFF Do Not Track integration are incomplete; and
- the settings and vault workflows still need full end-to-end coverage.

See the
[MV3 implementation backlog](https://github.com/BradGroux/AdNauseam/blob/mv3-improvement/platform/mv3/ADN-MV3-TODO.md)
for the remaining
work. Do not represent this branch as a production-ready AdNauseam release.

## Prerequisites

- Git with submodule support
- Bash
- Make
- Node.js 17.5 or later
- `jq`
- network access during the first ruleset build

The commands below have been verified on macOS. The build scripts are intended
for a Unix-like shell.

## Clone

Clone the MV3 branch and its submodules:

```bash
git clone \
  --branch mv3-improvement \
  --recurse-submodules \
  https://github.com/BradGroux/AdNauseam.git
cd AdNauseam
```

If the repository was cloned without submodules:

```bash
git submodule update --init --recursive
```

## Build for Chromium

```bash
make mv3-chromium
```

The unpacked extension is written to:

```text
dist/build/ADNLite.chromium
```

The build generates declarative rulesets and caches downloaded filter-list data
under `dist/build/mv3-data`. To force a fresh filter-list download:

```bash
make cleanassets
make mv3-chromium
```

## Install in Chrome developer mode

1. Open `chrome://extensions`.
2. Enable **Developer mode**.
3. Select **Load unpacked**.
4. Choose `dist/build/ADNLite.chromium`.
5. Open the extension details and confirm the ID is
   `dkoaabhijcomjinndlgbmfnmnjnmdeeb`.

Do not delete or move the selected build directory while Chrome is using it.

## Update an existing unpacked installation

Back up or export important AdNauseam data first. Then:

1. Build the new version.
2. Replace the contents of the folder already loaded by Chrome with the
   contents of `dist/build/ADNLite.chromium`.
3. Open `chrome://extensions`.
4. Select **Update**, or select **Reload** on the AdNauseam card.
5. Confirm the extension ID remains
   `dkoaabhijcomjinndlgbmfnmnjnmdeeb`.

Replacing the contents in place is preferable to loading a second folder
because it keeps Chrome pointed at the same unpacked source location.

## Verify the build

Confirm the generated manifest contract:

```bash
jq -e '
  .manifest_version == 3 and
  .background.service_worker == "/js/background.js" and
  .key == "ilkggpgmkemaniponkfgnkonpajankkm"
' dist/build/ADNLite.chromium/manifest.json
```

Check every generated JSON and JavaScript file:

```bash
find dist/build/ADNLite.chromium -type f -name '*.json' -print0 |
  xargs -0 -n 1 jq -e . >/dev/null

find dist/build/ADNLite.chromium -type f \
  \( -name '*.js' -o -name '*.mjs' \) -print0 |
  xargs -0 -n 1 node --check >/dev/null
```

Then perform a manual smoke test:

1. Load the unpacked extension without manifest or service-worker errors.
2. Open the popup and settings.
3. Visit a test page containing ads.
4. Confirm ads are hidden and collection counters change.
5. Confirm collected ads appear in the popup or vault.
6. Leave Chrome idle long enough to verify a queued visit attempt.

Record any failed step against the
[MV3 implementation backlog](https://github.com/BradGroux/AdNauseam/blob/mv3-improvement/platform/mv3/ADN-MV3-TODO.md).

## Other MV3 targets

The Makefile also exposes `mv3-edge`, `mv3-firefox`, and `mv3-safari` targets.
This document only claims verification for the Chromium target.

## Upstream

This work is based on the public
[`dhowe/AdNauseam` `mv3-improvement` branch](https://github.com/dhowe/AdNauseam/tree/mv3-improvement).
Keep the upstream remote when developing so fixes can be compared or proposed
back to the original project:

```bash
git remote add upstream https://github.com/dhowe/AdNauseam.git
git fetch upstream
```
