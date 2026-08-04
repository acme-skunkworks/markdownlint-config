---
title: "Migrate the release pipeline from Changesets to release-please"
release_note: "Infrastructure-only patch: the release pipeline moved from Changesets to release-please with a dated changelog subsystem; no preset rule changes."
version: "2.0.1"
created_at: "2026-06-23T19:28:27Z"
merged_at: "2026-06-23T19:28:27Z"
category: chore
breaking: false
issues:
  - A-379
---

## Changed

- Flipped the release pipeline from Changesets to **release-please**
  (`skip-changelog`) and added the dated `changelog/` subsystem plus the
  infrastructure TS toolchain (A-379).
- Rewrote `/send-it`, `CLAUDE.md`, and the ADRs for the release-please flow, and
  hardened checkout credential persistence and the pre-push bypass.

## Fixed

- The release **Pack** step no longer captures husky output as the tarball name;
  it now packs into an isolated directory and asserts a single artifact.
- Changelog validation accepts millisecond-precision ISO timestamps.

_Backfilled entry ([A-815](https://linear.app/rheged-studio/issue/A-815) /
[A-619](https://linear.app/rheged-studio/issue/A-619)): authored after the fact
to reconcile the changelog with the existing `v2.0.1` tag + GitHub Release. No
changes to the published markdownlint preset itself — infrastructure only._
