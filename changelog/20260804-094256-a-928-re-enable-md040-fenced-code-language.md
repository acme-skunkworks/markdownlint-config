---
title: Re-enable MD040 fenced-code-language requirement
release_note: Fenced code blocks must declare a language tag; untagged samples use text, console, or plaintext.
created_at: "2026-08-04T09:42:56Z"
merged_at:
branch: a-928-re-enable-md040-fenced-code-language-the-one-tightening
pr:
commit:
merge_strategy:
author: rob@acmeskunkworks.io
co_authors: []
category: feature
breaking: true
issues:
  - A-928
stats:
  files_changed:
  loc_added:
  loc_removed:
  commits:
---

## Breaking

- **MD040 is now enabled** in the shared `.markdownlint.jsonc`. Every fenced code
  block must carry a language tag. On upgrade, consumers with untagged fences
  will fail markdownlint until those blocks are labelled (`text`, `console`, or
  `plaintext` for language-less samples).

## Changed

- Re-enabled MD040 with a rewritten rationale documenting the
  `text` / `console` / `plaintext` convention ([A-928](https://linear.app/acme-skunkworks/issue/A-928)); bumped the audit date.
- Tagged the language-less tree listing in `infrastructure/README.md` as `text`.
