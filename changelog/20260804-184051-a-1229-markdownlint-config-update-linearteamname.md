---
title: Update Linear identity to Rheged Studio
release_note: ''
created_at: '2026-08-04T18:40:51Z'
merged_at: ''
branch: a-1229-markdownlint-config-update-linearteamname
pr:
commit: ''
author: rob@acmeskunkworks.io
co_authors: []
category: chore
breaking: false
issues:
  - A-1229
stats:
  files_changed:
  loc_added:
  loc_removed:
  commits:
---

## Changed

- Set vendored skill `linearTeamName` to `Rheged Studio` and
  `linearWorkspaceSlug` to `rheged-studio` in both `.claude/skills` and
  `.agents/skills` via `initialise.mjs --set` ([A-1229](https://linear.app/rheged-studio/issue/A-1229)).
- Rewrote committed `linear.app/acme-skunkworks` URLs to `rheged-studio` in
  eleven changelog entries and `docs/adr/0002-changesets-to-conventional-commits.md`
  (URL-only; ADR wording untouched).
