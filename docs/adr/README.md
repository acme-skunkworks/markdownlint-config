# Architecture Decision Records

This directory records architectural and infrastructure decisions for `@acme-studio/markdownlint-config`.

## Convention

ADRs follow [Michael Nygard's format](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions): a short markdown file capturing the context, decision, and consequences of a single choice. Files are numbered sequentially as `NNNN-kebab-case-title.md` (four digits, starting at `0001`). Numbers are never reused; superseded ADRs are linked from their replacement rather than deleted.

## Lifecycle states

Each ADR carries one of these statuses in its frontmatter:

- **Proposed** — drafted, awaiting acceptance.
- **Accepted** — agreed and executed (or actively executing).
- **Superseded by NNNN** — replaced by a later ADR; the link points forward.
- **Deprecated** — no longer relevant, kept for history.

## Index

- [0001 — Migrate CI from GitHub Actions to CircleCI](./0001-migrate-ci-from-github-actions-to-circleci.md) — _Superseded by 0002_
- [0002 — Changesets → Conventional Commits (release-please)](./0002-changesets-to-conventional-commits.md) — _Accepted_
