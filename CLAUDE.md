# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo

Standalone home for `@robeasthope/markdownlint-config` (extracted from `RobEasthope/protomolecule` — see `MIGRATION_FROM_PROTOMOLECULE.md`). Single shared markdownlint configuration. **The published artifact is `.markdownlint.json` itself** — there is no build step, no compiled output, no `dist/`. Consumers extend the JSON directly via `markdownlint-cli2`.

## Commands

```bash
pnpm install         # install deps + install husky hooks
pnpm run lint        # markdownlint on this repo's own .md files (delegates to lint:md)
pnpm run lint:md     # markdownlint-cli2 against **/*.{md,mdx}
pnpm run lint:md:fix # auto-fix
pnpm run lint:yaml   # yamllint . (semantic YAML check; warnings non-blocking)
pnpm run lint:workflows # actionlint on .github/workflows/
pnpm run format      # prettier write
pnpm changeset       # interactive changeset (or write .changeset/<slug>.md by hand)
```

Node 22 required (`.nvmrc`, `engines.node: ">=22"`, `engine-strict=true` in `.npmrc`).

## Local hooks

`pnpm install` runs `prepare` (`husky`), which installs the hooks under `.husky/`. Three hooks fire:

- **`pre-commit`** — runs `pnpm lint-staged`. Auto-fixes only the staged files: `prettier --write` for everything, `sort-package-json` for `**/package.json`, `markdownlint-cli2 --fix` for `**/*.{md,mdx}`, `yamllint` (read-only check) for `**/*.{yml,yaml}`, `actionlint` (read-only check) for `.github/workflows/*.{yml,yaml}`. Each task is wrapped in `bash -c '… "$@" --` so the staged file paths are passed through. The auto-fixers carry an `|| true` fallback so they never block — CI is the gate. The two YAML linters intentionally do **not** carry the `|| true` fallback: semantic errors block the commit (warnings don't). yamllint and actionlint are best-effort: if the tool isn't on `PATH` locally, the hook prints a platform-appropriate `brew install …` (or `pip` / `curl`) hint and skips. CI still enforces.
- **`commit-msg`** — strips any `Co-Authored-By: Claude … <noreply@anthropic.com>` trailer. Backstops the global `~/.claude/CLAUDE.md` rule (Claude is tooling, not a contributor).
- **`pre-push`** — blocks direct pushes to `main`; humans should use `/send-it` to open a PR. Bot users (`github-actions[bot]`, `road-runner-bot[bot]`) and the changesets release commit (`release: version packages`) bypass.

Hooks are dormant in CI: `release.yml` and `ci.yml` set `HUSKY=0` so the `prepare` script no-ops during `pnpm install`.

To bypass any hook in an emergency: `git commit --no-verify` or `git push --no-verify` — not recommended.

## Validating workflows and YAML

Two non-Node tools augment Prettier's formatting pass with the semantic checks Prettier can't see (Actions schema, `${{ … }}` expression typos, duplicate keys, etc.):

- **`actionlint` v1.7.5** — Go binary. Local install: `brew install actionlint` (macOS) or `bash <(curl -fsSL https://raw.githubusercontent.com/rhysd/actionlint/v1.7.5/scripts/download-actionlint.bash)` elsewhere. CI downloads the official tarball and caches it.
- **`yamllint` 1.37.1** — Python tool. Local install: `brew install yamllint` (macOS) or `pip install --user yamllint==1.37.1` elsewhere. CI installs via pip and caches `~/.local`.

Configuration: `.yamllint.yml` at the repo root extends defaults, demotes line-length / indentation to warnings (Prettier owns formatting), allows the GitHub Actions truthy values (`on`, `off`, `yes`, `no`), and ignores `node_modules/`, `dist/`, `.turbo/`, `pnpm-lock.yaml`. No `.actionlintrc.yaml` — defaults are fine for this repo.

Enforcement: pre-commit is best-effort (skip with install hint when missing); CI is the `yaml-lint` job in `ci.yml`, parallel to `build-and-lint`, always enforced.

## Release workflow

There are two release modes — know which one you're in.

### Day-to-day releases (CI via OIDC)

Once the package's Trusted Publisher mapping points at this repo's `release.yml`, every release flows through CI:

1. Make changes on a feature branch; `/send-it` bundles, writes `.changeset/<slug>.md`, pushes, opens a PR. CI (`.github/workflows/ci.yml`) runs lint/changeset-status on the PR.
2. After merge, `changesets/action` (`.github/workflows/release.yml`) opens a "release: version packages" PR; the post-step retitles it to `@robeasthope/markdownlint-config@<version>`.
3. Merging that PR triggers publish: npm via OIDC Trusted Publishing (no token, no OTP) + provenance attestation on the npm artifact. **No GitHub Packages publish** — this package is npm-only.

Don't reintroduce `NPM_TOKEN` **as a CI secret** unless OIDC is verified broken. The local `.env`-based `NPM_TOKEN` is a different concern — it's for laptop-driven publishes only, never CI.

**Manual changeset.** `pnpm changeset` (interactive) or hand-write `.changeset/<slug>.md`:

```markdown
---
"@robeasthope/markdownlint-config": <patch|minor|major>
---

<body>
```

### Manual publish (CI fallback, after the package exists)

For when CI is down. Auth setup (one-time, or after rotating your token):

```bash
NPM_TOKEN=$(grep '^NPM_TOKEN=' .env | cut -d'=' -f2-)
npm config set //registry.npmjs.org/:_authToken "$NPM_TOKEN"
npm whoami    # verify
```

The token must be a **Granular Access Token with the "Bypass 2FA" option enabled at creation time**. Without that flag, every publish hits `EOTP` and you're stuck. Tokens are immutable after creation — if you forgot the flag, revoke and regenerate.

Then publish:

```bash
pnpm run release:manual:dry    # simulate — verifies tarball + auth
pnpm run release:manual        # actual publish
```

`--provenance=false` is intentional — provenance attestation requires a GitHub Actions OIDC issuer, which a laptop doesn't have. Manual publishes ship without the provenance badge; CI publishes get it.

Don't try `pnpm run release:manual -- --dry-run`. The chained-script + `--` separator confuses npm into treating `--dry-run` as a positional package spec. Use `release:manual:dry`.

## Bootstrap publish — read this when setting up a new package

> **Not directly applicable to this repo.** `@robeasthope/markdownlint-config` already exists on npm (last published v1.1.1 from protomolecule), so the only manual step here is updating the Trusted Publisher mapping on `npmjs.com/package/@robeasthope/markdownlint-config/access` to point at this repo's `release.yml`. The notes below are kept verbatim as institutional knowledge for the next sibling extraction (e.g. the next package pulled out of protomolecule).

The very first publish of a brand-new npm package **cannot go through CI**. Two reasons that compound:

- npm (unlike PyPI) has no pending-Trusted-Publisher flow. The package must exist on the registry before the Trusted Publisher form is reachable at `npmjs.com/package/<name>/access`.
- npm enforces 2FA at the publish endpoint for the first publish of a new package, irrespective of account/org/token bypass settings. Granular bypass-2FA tokens only honour the bypass on subsequent publishes.

So bootstrap is always: manual first publish → configure Trusted Publisher → CI takes over from publish #2.

**Pre-flight:**

- You belong to the target npm org with publish rights.
- npm CLI ≥ 11.5.1 (`npm install -g npm@latest`).
- Account has 2FA enabled with **recovery codes generated and saved** (you'll need one).
- `package.json` is at the version you want to ship (`pnpm changeset version` consumes pending changesets and bumps).

**Sequence:**

1. `pnpm changeset version` — consume pending changesets, bump `package.json`, write `CHANGELOG.md`.
2. `pnpm run release:manual:dry` — verify tarball + auth. **Note:** dry-run does NOT trigger 2FA enforcement, so a successful dry-run does not predict a successful real publish. It only proves the tarball is valid and your token authenticates.
3. `pnpm run release:manual` — first real attempt. **This will fail with `EOTP`.** That's expected.
4. Use a **recovery code as the `--otp` value**:

   ```bash
   npm publish --access public --provenance=false --otp=<recovery-code>
   ```

   Generate codes at npmjs.com → Profile → Two-Factor Authentication → Manage Recovery Codes. Each is single-use. The format is a long hex string (not a 6-digit TOTP) — npm accepts it as `--otp` anyway.

5. After publish succeeds, **immediately regenerate recovery codes**. The one you used is burnt; if you transmitted it anywhere (chat, paste buffer with cloud sync, screen share), treat the rest of the set as compromised.
6. Configure Trusted Publisher: `https://www.npmjs.com/package/<name>/access` → GitHub Actions → org, repo, workflow filename (`release.yml`), environment blank.
7. From here on, releases go through CI cleanly.

### Things that look like solutions but aren't

Saving these to spare the next bootstrap from rediscovering them:

- `npm publish --auth-type=web` — flag is for `npm login`, ignored by `publish`.
- Toggling "Require 2FA for write actions" off in account settings.
- Disabling org-level 2FA enforcement.
- Generating a Granular token with bypass-2FA enabled — works for publish #2+, NOT publish #1.
- `npm login --auth-type=web` to refresh the session token. Auth swaps successfully but the publish endpoint still demands OTP.
- `oathtool` for generating TOTP — only works if you have a TOTP secret, and **npm has phased TOTP out of new accounts** (only passkeys + recovery codes are offered now).
- Disabling 2FA entirely — npm's policy _requires_ either 2FA or a bypass-2FA token; you can't disable both. And the bypass token doesn't help for publish #1 anyway.

Recovery codes are the answer because they're the only OTP-shaped value an npm account can produce when its only 2FA factor is a passkey.

## OIDC Trusted Publisher

The Trusted Publisher mapping on npm is what makes the OIDC publish work without an `NPM_TOKEN`. After this repo's first CI publish:

- `@robeasthope/markdownlint-config`'s Trusted Publisher must point at `acme-skunkworks/markdownlint-config` → workflow `release.yml`, environment blank.
- The mapping was previously configured against `RobEasthope/protomolecule` (when the package shipped from there). It must be updated **before** the first CI publish from this repo, or the publish step fails with an OIDC mismatch.
- Update path: log into npmjs.com → `npmjs.com/package/@robeasthope/markdownlint-config/access` → GitHub Actions section → edit the publisher.
