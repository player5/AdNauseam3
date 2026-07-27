# AdNauseam Lite for Chrome Manifest V3

This public fork packages the experimental Manifest V3 port of
[AdNauseam](https://adnauseam.io) for current versions of Google Chrome.

Chrome removed Manifest V2 support after Chrome 138, so the existing AdNauseam
Chromium release cannot run in current Chrome even when developer mode is
enabled. This fork tracks the upstream `mv3-improvement` work and makes it
practical to build and load the development version locally.

> [!WARNING]
> This is a development build, not a stable AdNauseam release. The core
> collection, storage, and visit backend is largely implemented, but several
> workflows still need production-grade persistence and end-to-end coverage.

## What this fork changes

- Preserves the established Chromium extension ID
  `dkoaabhijcomjinndlgbmfnmnjnmdeeb`.
- Fixes page-specific cosmetic filtering that previously failed with
  `ReferenceError: selectors is not defined`.
- Documents the real AdNauseam Lite build, developer-mode installation, update,
  verification, and known limitations.
- Makes `mv3-improvement` the default branch so the Chrome-compatible work is
  visible from the repository landing page.

## Quick start

Prerequisites: Git, Bash, Make, Node.js 17.5 or later, `jq`, and network access
for the first ruleset build.

```bash
git clone \
  --branch mv3-improvement \
  --recurse-submodules \
  https://github.com/BradGroux/AdNauseam.git
cd AdNauseam
make mv3-chromium
```

The unpacked extension is generated at:

```text
dist/build/ADNLite.chromium
```

To load it:

1. Open `chrome://extensions`.
2. Enable **Developer mode**.
3. Select **Load unpacked**.
4. Choose `dist/build/ADNLite.chromium`.
5. Confirm the extension ID is
   `dkoaabhijcomjinndlgbmfnmnjnmdeeb`.

For an existing unpacked installation, replace its files in place and select
**Update** or **Reload** from `chrome://extensions`.

## Current limitations

Known incomplete areas include:

- persistent thumbnails for collected ads;
- visit scheduling across service-worker suspension;
- idle-aware visit scheduling;
- notifications and EFF Do Not Track integration; and
- full settings and vault workflow coverage.

Do not publish this branch as a stable AdNauseam release without completing the
remaining implementation and browser testing.

## Documentation

- [Complete Chromium MV3 build and installation guide](MV3-CHROMIUM.md)
- [MV3 implementation backlog](platform/mv3/ADN-MV3-TODO.md)
- [Issue #1: extension identity and cosmetic-filter fix](https://github.com/BradGroux/AdNauseam/issues/1)
- [PR #2: verified implementation](https://github.com/BradGroux/AdNauseam/pull/2)

## Verification baseline

The current fork has been checked with:

- a clean `make mv3-chromium` build;
- generated-manifest contract validation;
- parsing all generated JSON files with `jq`;
- parsing all generated JavaScript files with `node --check`; and
- a focused runtime test for page-specific cosmetic-filter injection.

See [MV3-CHROMIUM.md](MV3-CHROMIUM.md#verify-the-build) for reproducible
verification commands and the manual smoke-test checklist.

## Upstream and license

AdNauseam was created by Daniel C. Howe, Helen Nissenbaum, Mushon Zer-Aviv, and
the project contributors. This fork is based on
[`dhowe/AdNauseam`](https://github.com/dhowe/AdNauseam), particularly its
[`mv3-improvement`](https://github.com/dhowe/AdNauseam/tree/mv3-improvement)
branch.

AdNauseam is built on [uBlock Origin](https://github.com/gorhill/uBlock).
Do not run it alongside another similarly purposed blocker.

Licensed under GPLv3. See [LICENSE.txt](LICENSE.txt).
