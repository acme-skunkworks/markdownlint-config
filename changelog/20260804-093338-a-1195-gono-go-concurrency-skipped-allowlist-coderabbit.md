---
title: Stop GO/NO GO false-reds and CodeRabbit PR-description edits
release_note: ""
created_at: "2026-08-04T09:33:38Z"
merged_at:
branch: a-1195-gono-go-concurrency-skipped-allowlist-coderabbit
pr:
commit:
author: rob@acmeskunkworks.io
co_authors: []
category: chore
breaking: false
issues:
  - A-1195
stats:
  files_changed:
  loc_added:
  loc_removed:
  commits:
---

## Changed

**GO/NO GO concurrency ([A-1100](https://linear.app/acme-skunkworks/issue/A-1100) / [A-1195](https://linear.app/acme-skunkworks/issue/A-1195))** — set `cancel-in-progress: false` on `ci.yml` and `validate-pr-title.yml` so a superseded run cannot mint a false-red gate.

**GO/NO GO verdict allowlist ([A-1103](https://linear.app/acme-skunkworks/issue/A-1103) / [A-1195](https://linear.app/acme-skunkworks/issue/A-1195))** — accept `skipped` only on `release-please--*` branches; everywhere else every needed job must report `success`.

**CodeRabbit walkthrough summary ([A-1102](https://linear.app/acme-skunkworks/issue/A-1102) / [A-1195](https://linear.app/acme-skunkworks/issue/A-1195))** — set `high_level_summary_in_walkthrough: true` so the high-level summary lands in the walkthrough comment instead of rewriting the PR description (and re-firing CI).
