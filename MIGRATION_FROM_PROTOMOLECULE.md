# Migration from protomolecule

This repo is the new home for the markdownlint configuration previously shipped from `RobEasthope/protomolecule` as `@robeasthope/markdownlint-config`. The package is **renamed** to `@acme-skunkworks/markdownlint-config` for this release — both the publish source and the npm scope move. Consumers must update their dependency.

## Source

- **Source repo:** `RobEasthope/protomolecule`
- **Source path:** `packages/markdownlint-config/`
- **Source SHA:** `26a77851e5df47c9f6db6690f33923bf42a1901b`
- **Last commit touching the source:** `a0f4f6e8687a0df831cad54370dc84f89a70f51d` (release: version packages for release, 2025-11-18)
- **v1.1.1 publish commit:** `e98d3f7fdaee497ae24a06df75389273d5e0428d` (fix(markdownlint-config): disable MD040 rule, 2025-11-17)
- **Last published version under the old name/origin:** `@robeasthope/markdownlint-config@1.1.1`
- **First version published under the new name/origin:** `@acme-skunkworks/markdownlint-config@2.0.0` (major bump signals the breaking rename; configuration bytes are byte-identical to the predecessor's v1.1.1)
- **Port date:** 2026-05-08
- **Tracking ticket:** [ASW-91](https://linear.app/goose-and-hobbes/issue/ASW-91)

## What changed in the port

The port is a clean copy of the source files plus the standalone-repo infrastructure (workflows, hooks, send-it tooling, changesets) cloned from the sibling `acme-skunkworks/eslint-config`. Edits below are deliberate.

### Package identity

- `name`: **renamed** from `@robeasthope/markdownlint-config` → `@acme-skunkworks/markdownlint-config`. On npm this is a brand-new package; consumers must update their dependency name and any `extends` references inside their `.markdownlint-cli2.jsonc` / `.markdownlint.json`. The first release from this repo is a major bump (1.1.1 → 2.0.0) to signal the rename.
- The legacy `@robeasthope/markdownlint-config@1.1.1` continues to exist on npm and remains installable, but receives no further updates from this repo. It can be deprecated via `npm deprecate` once consumers have migrated (out of scope for this PR).
- `repository.url`, `homepage`, `bugs.url`: rewritten from `RobEasthope/protomolecule` (with `directory: "packages/markdownlint-config"`) to `acme-skunkworks/markdownlint-config`. `repository.directory` is dropped since the package is no longer a sub-path of a monorepo.
- `author` is unchanged (still Rob Easthope).

### Trusted Publisher setup (fresh)

Because `@acme-skunkworks/markdownlint-config` is a brand-new package on npm, the Trusted Publisher form at `npmjs.com/package/@acme-skunkworks/markdownlint-config/access` is **unreachable until the package exists**. The bootstrap is therefore: manual first publish (recovery-code OTP path — see `CLAUDE.md` "Bootstrap publish") → configure Trusted Publisher → CI takes over from publish #2.

Target Trusted Publisher mapping after bootstrap:

- **Repository:** `acme-skunkworks/markdownlint-config`
- **Workflow:** `release.yml`
- **Environment:** _(blank)_

The legacy `@robeasthope/markdownlint-config` package on npm has its own Trusted Publisher mapping pointing at `RobEasthope/protomolecule`. That mapping is independent of this repo and stays as-is — leave it alone.

### `package.json` deltas (vs. the protomolecule source)

| Field                        | Old (protomolecule)                                   | New (this repo)                                                                   |
| ---------------------------- | ----------------------------------------------------- | --------------------------------------------------------------------------------- |
| `name`                       | `@robeasthope/markdownlint-config`                    | `@acme-skunkworks/markdownlint-config`                                            |
| `version`                    | `1.1.1`                                               | `1.1.1` _(in source)_ → `2.0.0` _(after `pnpm changeset version`)_                |
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
- `files: [".markdownlint.json", "README.md"]` is unchanged. The published tarball contains the same two files as the predecessor; only the package name on the registry differs.

### Preserved verbatim

- `.markdownlint.json` — the 13-key config (including `MD040: false` from v1.1.1) copied with no edits.
- `.markdownlint-cli2.jsonc` — `extends: "./.markdownlint.json"` self-test, unchanged.
- `CHANGELOG.md` — copied with the top-of-file heading retitled to `# @acme-skunkworks/markdownlint-config`. Legacy v1.0.0–v1.1.1 entries below preserve the original `@robeasthope/markdownlint-config` references in their bodies (they describe the predecessor as it shipped), and protomolecule commit hashes / PR links stay as historical context. The new v2.0.0 entry will be prepended by `pnpm changeset version` from the changeset on this branch.
- `LICENSE` — copied from `RobEasthope/protomolecule/LICENSE` (MIT, Copyright (c) 2025 Rob Easthope). Previously inherited from the monorepo root; now lives in the package root.
- `README.md` — copied with three edits: (1) all install / `extends` examples reference `@acme-skunkworks/markdownlint-config` (the new package name); (2) the `[LICENSE](../../LICENSE)` link points at `./LICENSE` (since LICENSE now sits in the same dir) and the "Package Structure" tree was flattened to drop the `packages/markdownlint-config/` prefix; (3) rule documentation was corrected — the old README claimed MD040 was enabled, but v1.1.1 actually disabled it, and the README now reflects the actual config.

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
- `send-it.md`: changeset frontmatter example uses `@acme-skunkworks/markdownlint-config` (not the eslint-config template's `@acme-skunkworks/eslint-config`).

## First-publish sequence (post-merge)

The first publish from this repo is a **manual bootstrap** because `@acme-skunkworks/markdownlint-config` is brand-new on npm. npm enforces 2FA at the publish endpoint for the first publish of a new package regardless of token settings, and the Trusted Publisher form isn't reachable until the package exists. Recovery codes are the workaround (full rationale in `CLAUDE.md` "Bootstrap publish").

1. **Land this PR.** Includes a `major` changeset on disk (consumed locally next, not by CI).
2. On `main` locally, create a release branch off the merged HEAD:

   ```bash
   git checkout main && git pull
   git checkout -b release/2.0.0
   pnpm changeset version    # bumps package.json 1.1.1 → 2.0.0, writes CHANGELOG.md, deletes the changeset file
   git add package.json CHANGELOG.md .changeset/
   git commit -m "release: version packages"
   ```

3. Authenticate with npm using the legacy auth flow (avoids the macOS passkey path that doesn't reliably surface a TOTP prompt):

   ```bash
   npm login --auth-type=legacy
   #   OTP: paste a recovery code (long hex string from npmjs.com → Profile → 2FA → Manage Recovery Codes)
   npm whoami    # verify
   ```

4. Dry-run, then publish:

   ```bash
   pnpm run release:manual:dry
   pnpm run release:manual
   #   If npm prompts for OTP again, paste a FRESH recovery code (each is single-use).
   #   If non-TTY swallows the prompt, fall back to: npm publish --access public --provenance=false --otp=<recovery-code>
   ```

5. **Immediately regenerate recovery codes** at npmjs.com → Profile → 2FA → Manage Recovery Codes. The codes you used are burnt; treat the rest of the set as compromised if any of them touched a clipboard with cloud sync.
6. Configure Trusted Publisher: `npmjs.com/package/@acme-skunkworks/markdownlint-config/access` → GitHub Actions → repo `acme-skunkworks/markdownlint-config`, workflow `release.yml`, environment blank.
7. Push the release branch and open a housekeeping PR for the version bump (`release: @acme-skunkworks/markdownlint-config@2.0.0`). Merge.
8. From here on, releases ride CI/OIDC: `/send-it` writes a changeset, the changesets/action workflow on `main` opens a version PR, merging the version PR triggers a publish with provenance attestation.
