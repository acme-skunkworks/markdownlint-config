# Migration from protomolecule

This repo is the new home for the markdownlint configuration previously shipped from `RobEasthope/protomolecule` as `@robeasthope/markdownlint-config`. The npm package name is **unchanged** — only the publish source moves.

## Source

- **Source repo:** `RobEasthope/protomolecule`
- **Source path:** `packages/markdownlint-config/`
- **Source SHA:** `26a77851e5df47c9f6db6690f33923bf42a1901b`
- **Last commit touching the source:** `a0f4f6e8687a0df831cad54370dc84f89a70f51d` (release: version packages for release, 2025-11-18)
- **v1.1.1 publish commit:** `e98d3f7fdaee497ae24a06df75389273d5e0428d` (fix(markdownlint-config): disable MD040 rule, 2025-11-17)
- **Last published version under the old origin:** `@robeasthope/markdownlint-config@1.1.1`
- **Port date:** 2026-05-08
- **Tracking ticket:** [ASW-91](https://linear.app/goose-and-hobbes/issue/ASW-91)

## What changed in the port

The port is a clean copy of the source files plus the standalone-repo infrastructure (workflows, hooks, send-it tooling, changesets) cloned from the sibling `acme-skunkworks/eslint-config`. Edits below are deliberate.

### Package identity

- `name`: **unchanged** at `@robeasthope/markdownlint-config`. This is the same npm package — consumers do not need to update their `package.json`. The first release from this repo will be a patch bump (1.1.1 → 1.1.2).
- `repository.url`, `homepage`, `bugs.url`: rewritten from `RobEasthope/protomolecule` (with `directory: "packages/markdownlint-config"`) to `acme-skunkworks/markdownlint-config`. `repository.directory` is dropped since the package is no longer a sub-path of a monorepo.
- `author` is unchanged (still Rob Easthope).

### Why the GitHub org differs from the npm scope

The GitHub repo lives under `acme-skunkworks/` (matching the sibling `acme-skunkworks/eslint-config`) while the npm package keeps Rob's personal scope (`@robeasthope/...`). This is intentional — the npm package name is part of the public API and changing it would break consumers; the GitHub org is internal plumbing and the standalone-repo template is org-scoped.

### Trusted Publisher remap

The npm Trusted Publisher mapping for `@robeasthope/markdownlint-config` must be updated **before** the first CI publish from this repo:

- **Old:** `RobEasthope/protomolecule` → `release.yml`
- **New:** `acme-skunkworks/markdownlint-config` → `release.yml`

Without this update, the first OIDC publish from this repo will fail with an issuer mismatch. Update path: log into npmjs.com → `npmjs.com/package/@robeasthope/markdownlint-config/access` → GitHub Actions section → edit publisher.

### `package.json` deltas (vs. the protomolecule source)

| Field                        | Old (protomolecule)                                   | New (this repo)                                                                   |
| ---------------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------------- |
| `repository.url`             | `https://github.com/RobEasthope/protomolecule.git`    | `https://github.com/acme-skunkworks/markdownlint-config.git`                      |
| `repository.directory`       | `packages/markdownlint-config`                        | _(dropped)_                                                                       |
| `homepage`                   | `https://github.com/RobEasthope/protomolecule#readme` | `https://github.com/acme-skunkworks/markdownlint-config#readme`                   |
| `bugs.url`                   | `https://github.com/RobEasthope/protomolecule/issues` | `https://github.com/acme-skunkworks/markdownlint-config/issues`                   |
| `packageManager`             | _(not set; inherited from monorepo root)_             | `pnpm@10.33.0`                                                                    |
| `engines.node`               | _(not set)_                                           | `>=22`                                                                            |
| `scripts.prepare`            | _(not present)_                                       | `husky`                                                                           |
| `scripts.lint`               | _(not present)_                                       | `pnpm run lint:md`                                                                |
| `scripts.lint:yaml`          | _(not present)_                                       | `yamllint .`                                                                      |
| `scripts.lint:workflows`     | _(not present)_                                       | `actionlint`                                                                      |
| `scripts.version`            | _(not present)_                                       | `changeset version`                                                               |
| `scripts.release`            | _(not present)_                                       | `changeset publish`                                                               |
| `scripts.release:manual`     | _(not present)_                                       | `npm publish --access public --provenance=false`                                  |
| `scripts.release:manual:dry` | _(not present)_                                       | same with `--dry-run`                                                             |
| `lint-staged`                | _(not present)_                                       | full block (prettier, markdownlint-cli2, sort-package-json, yamllint, actionlint) |
| `devDependencies` adds       | —                                                     | `@changesets/cli`, `husky`, `lint-staged`, `prettier`, `sort-package-json`        |

**Dropped fields:** none from protomolecule's package.json. (No `prepublishOnly` to drop — there was none to begin with, and we intentionally don't add one because there's no build step to gate.)

### Build / tooling

- **No build step.** The published artifact is `.markdownlint.json` itself — consumers `extend` it directly. There is no `tsc`, no `dist/`, no `prepublishOnly`. This differs from the eslint-config sibling, which compiles TypeScript.
- `files: [".markdownlint.json", "README.md"]` is unchanged — the package contents on npm stay the same.

### Preserved verbatim

- `.markdownlint.json` — the 13-key config (including `MD040: false` from v1.1.1) copied with no edits.
- `.markdownlint-cli2.jsonc` — `extends: "./.markdownlint.json"` self-test, unchanged.
- `CHANGELOG.md` — copied verbatim. The protomolecule commit hashes / PR references stay as historical context (the issues remain readable on GitHub even after the protomolecule package is consolidated away).
- `LICENSE` — copied from `RobEasthope/protomolecule/LICENSE` (MIT, Copyright (c) 2025 Rob Easthope). Previously inherited from the monorepo root; now lives in the package root.
- `README.md` — copied with one edit: the `[LICENSE](../../LICENSE)` link points at `./LICENSE` (since LICENSE now sits in the same dir). The "Package Structure" tree was also flattened to drop the `packages/markdownlint-config/` prefix. Rule documentation was corrected: previous README claimed MD040 was enabled; v1.1.1 actually disabled it, and the README now reflects the actual config.

## Infrastructure cloned from `acme-skunkworks/eslint-config`

Per ASW-91 scope, the standalone-repo infrastructure was cloned from the sibling `acme-skunkworks/eslint-config`:

- **Workflows:** `ci.yml` (build-and-lint + yaml-lint, both gated against `changeset-release/*`), `release.yml` (Changesets via OIDC, with the npm self-upgrade workaround and the package@version retitle step), `claude.yml` (interactive `@claude` mentions), `claude-code-review.yml` (auto PR review).
- **Husky kit:** `pre-commit` (lint-staged), `commit-msg` (strip Claude trailers), `pre-push` (block direct pushes to `main`).
- **Send-it:** `.claude/commands/send-it.md` slash command + `scripts/send-it/derive-changeset.mjs` deterministic helper.
- **Changesets:** `.changeset/config.json` (commit: false, access: public), `.changeset/README.md`.
- **Configs:** `.yamllint.yml`, `.npmrc`, `.nvmrc` (Node 22), `.env.example`, `.gitignore`, `.prettierignore`.

### Deltas vs. the eslint-config template

- `ci.yml`: dropped the `🏗️ Build` step (no build for a JSON config).
- `release.yml`: dropped the "Setup Node.js (GitHub Packages)" + "Publish to GitHub Packages" steps. This package is npm-only.
- `package.json`: lint-staged and devDependencies have no ESLint entries (this repo doesn't lint TypeScript).
- `CLAUDE.md`: dropped the eslint-config-specific Architecture section (preset descriptions) and Editing rules section.
- `.gitignore`: dropped the TypeScript build entries (`*.tsbuildinfo`, `.eslintcache*`).
- `.prettierignore`: dropped the `tsconfig.json` line (no tsconfig).
- `send-it.md`: changeset frontmatter example uses `@robeasthope/markdownlint-config` (not the eslint-config template's `@acme-skunkworks/eslint-config`).

## First-publish sequence (post-merge)

1. **Land this PR.** Infra-only; no changeset.
2. **Update the Trusted Publisher mapping** on npmjs.com (manual web step — see "Trusted Publisher remap" above).
3. Open a follow-up PR adding a changeset (`patch` bump → 1.1.2). Merge.
4. Changesets opens a version PR titled `@robeasthope/markdownlint-config@1.1.2` (after the retitle step). Merge it.
5. CI publishes 1.1.2 to npm via OIDC. Verify on npmjs.com that provenance attestation points at `acme-skunkworks/markdownlint-config`.
