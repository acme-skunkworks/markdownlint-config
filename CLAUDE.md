# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo

Standalone home for `@acme-skunkworks/markdownlint-config` (extracted from `RobEasthope/protomolecule` — see `MIGRATION_FROM_PROTOMOLECULE.md`). Single shared markdownlint configuration, published from this repo via release-please (Conventional Commits — A-379). **The published artifact is `.markdownlint.jsonc` itself** — there is no build step, no compiled output, no `dist/`. Consumers extend the JSONC directly via `markdownlint-cli2`, whose parser stack handles the inline per-rule rationale comments. Bare `markdownlint` with `JSON.parse` would reject the comments — call this out if/when a non-cli2 consumer surfaces.

There _is_ a TypeScript toolchain in the repo (`infrastructure/` — the dated-changelog tooling), but it is **dev-only**: typechecked with `tsc --noEmit`, tested with vitest, and never published. It does not change the shippable surface.

## British English

Use British English in all prose: code comments, documentation (`README.md`, `CLAUDE.md`, ADRs, `MIGRATION_FROM_PROTOMOLECULE.md`), changelog entries, commit messages, PR titles/descriptions, and user-facing strings.

**Spelling:** _colour_, _behaviour_, _organisation_, _centre_, _catalogue_, _recognise_, _analyse_, _realise_, _optimise_, _customise_, _favourite_, _travelling_, _cancelled_, _modelling_, _labelled_. Use _programme_ for the broadcast/agenda sense; keep _program_ when referring to source code.

**Grammar/punctuation:** _whilst_ and _amongst_ are acceptable. Prefer single quotes for short quotations where appropriate; place full stops outside closing quotation marks when the quoted phrase is partial.

**Scope — do NOT rewrite:**

- Identifiers and APIs that mirror upstream names: CSS `color:` property, function/variable names, npm package names (`ansi-colors`, `supports-color`), CLI flags (`actionlint -color`), config keys.
- Quoted upstream text and dependency names.
- API field values that already use US spelling — for example, **Linear workflow state names like `Canceled`** come from the Linear API and stay as US-spelt when referenced in code, config, or docs.
- Third-party request/response shapes.

If in doubt about whether a token is identifier or prose, leave the original spelling.

## GitHub Actions repo config (A-176)

Non-secret knobs shared by `ci.yml` and `pkg-release.yml` live in **`infrastructure/repo-config.yaml`**, loaded at runtime by the composite `.github/actions/load-repo-config` (`uses: ./.github/actions/load-repo-config`).

| Key                         | Purpose                                                                                                                     |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `defaultBranch`             | Canonical default branch; keep in sync with static `on:` triggers (GitHub cannot derive `on.push.branches` from this file). |
| `nodeVersionFile`           | Passed to `actions/setup-node` `node-version-file`.                                                                         |
| `npmRegistryUrl`            | Public npm registry (`setup-node` when talking to npmjs).                                                                   |
| `npmScope`                  | Package scope; must equal the owning GitHub org so `setup-node` scopes `.npmrc` for the GitHub Packages leg (A-323).        |
| `githubPackagesRegistryUrl` | GitHub Packages npm registry (`https://npm.pkg.github.com`) — the secondary publish target (A-323).                         |

Secrets (`GITHUB_TOKEN`), OIDC Trusted Publishing, and release-please behaviour are unchanged — not in this file.

## Commands

```bash
pnpm install         # install deps + install husky hooks
pnpm run lint        # markdownlint on this repo's own .md files (delegates to lint:md)
pnpm run lint:md     # markdownlint-cli2 against **/*.{md,mdx}
pnpm run lint:md:fix # auto-fix
pnpm run lint:yaml   # yamllint . (semantic YAML check; warnings non-blocking)
pnpm run lint:workflows  # actionlint on .github/workflows/
pnpm run lint:sh     # shellcheck on infrastructure/scripts/*.sh + .husky/*
pnpm tsc             # type-check the infrastructure/ TS (tsc --noEmit; nothing is published)
pnpm test            # vitest run (infrastructure/tests/**/*.test.ts)
pnpm test:watch      # vitest in watch mode
pnpm test:sh         # bats on infrastructure/tests/*.bats
pnpm run format      # prettier write
pnpm validate:changelog  # validate dated changelog/ entries (CI: shared lint caller)
```

Node 22 required (`.nvmrc`, `engines.node: ">=22"`, `engine-strict=true` in `.npmrc`).

## Agent skills

The shipping and housekeeping commands are provided by the shared
[`@acme-skunkworks/agent-skills`](https://github.com/acme-skunkworks/agent-skills)
bundles, installed via [skills.sh](https://skills.sh) under `.claude/skills/`
(and mirrored to `.agents/skills/` for Cursor). They replace the previous bespoke
`.claude/commands/send-it.md`:

- `/send-it` — commit → preflight → dated changelog entry → Conventional-Commit PR
  title → push → draft PR → Linear → In Review. Delegates to `preflight`,
  `changelog`, and `linear-sync`.
- `/preflight` — change-gated, branch-scoped lint preflight.
- `/changelog` — author/refresh/validate the dated `changelog/` entry.
- `/linear-sync` — transition linked Linear issues.
- `/cleanup-repo` — prune merged branches, worktrees, filesystem cruft.
- `/triage-pr` — drive a PR from draft-with-failing-CI to merge-ready.

Each skill reads its own `config.json` (reconciled by `initialise-skills` from
this repo's facts). The repo's `lint:md` already excludes `.claude/skills/` and
`.agents/skills/`, so the vendored bundles are not linted. Re-install or upgrade
with `npx skills add … --copy`; re-run `initialise-skills` afterwards.

## Local hooks

`pnpm install` runs `prepare` (`husky`), which installs the hooks under `.husky/`. Three hooks fire:

- **`pre-commit`** — runs `pnpm lint-staged`. Auto-fixes only the staged files: `prettier --write` for everything, `sort-package-json` for `**/package.json`, `markdownlint-cli2 --fix` for `**/*.{md,mdx}`, `yamllint` (read-only check) for `**/*.{yml,yaml}`, `actionlint` (read-only check) for `.github/workflows/*.{yml,yaml}`. Each task is wrapped in `bash -c '… "$@" --` so the staged file paths are passed through. The auto-fixers carry an `|| true` fallback so they never block — CI is the gate. The two YAML linters intentionally do **not** carry the `|| true` fallback: semantic errors block the commit (warnings don't). yamllint and actionlint are best-effort: if the tool isn't on `PATH` locally, the hook prints a platform-appropriate `brew install …` (or `pip` / `curl`) hint and skips. CI still enforces.
- **`commit-msg`** — strips any `Co-Authored-By: Claude … <noreply@anthropic.com>` trailer. Backstops the global `~/.claude/CLAUDE.md` rule (Claude is tooling, not a contributor).
- **`pre-push`** — blocks direct pushes to `main`; humans should use `/send-it` to open a PR. Bot users (`github-actions[bot]`, `road-runner-bot[bot]`) bypass — which is what lets the release-please release commit (`chore(main): release <version>`, pushed by road-runner-bot) through. The bypass signal is the bot identity, not the commit message (a human-authored `chore(main): release …` does not bypass). It also runs `pnpm lint:yaml` + `pnpm lint:workflows` as a last-line gate (best-effort; skips with an install hint if the tool is missing).

Hooks are dormant in CI: `pkg-release.yml` and `ci.yml` set `HUSKY=0` so the `prepare` script no-ops during `pnpm install`.

To bypass any hook in an emergency: `git commit --no-verify` or `git push --no-verify` — not recommended.

## The published config (`.markdownlint.jsonc`)

`.markdownlint.jsonc` is the entire deliverable — a single shared markdownlint ruleset with inline per-rule rationale comments. `.markdownlint-cli2.jsonc` is this repo's own consumer of it (`extends: ./.markdownlint.jsonc` plus the glob list); changes to that wrapper are tooling, not shippable. When editing `.markdownlint.jsonc`:

- Preserve the inline comments — they document why each rule deviates from defaults, and the cli2 parser (jsonc) keeps them.
- The artifact must stay JSONC-parseable by `markdownlint-cli2`. Don't introduce syntax a strict `JSON.parse` consumer couldn't recover from without the cli2 stack — and if a non-cli2 consumer ever surfaces, flag the comment-stripping requirement.
- Re-audit against the pinned `markdownlint-cli2` version when bumping it (the file header records the last audit date).

## Validating workflows and YAML

Two non-Node tools augment Prettier's formatting pass with the semantic checks Prettier can't see (Actions schema, `${{ … }}` expression typos, duplicate keys, etc.):

- **`actionlint` v1.7.5** — Go binary. Local install: `brew install actionlint` (macOS) or `bash <(curl -fsSL https://raw.githubusercontent.com/rhysd/actionlint/v1.7.5/scripts/download-actionlint.bash)` elsewhere. CI downloads the official tarball and caches it.
- **`yamllint` 1.37.1** — Python tool. Local install: `brew install yamllint` (macOS) or `pip install --user yamllint==1.37.1` elsewhere. CI installs via pip and caches `~/.local`.

**Digest-pinned bootstraps (A-327).** The CI install scripts for these tools fetch-and-execute third-party code, so each is pinned by digest, not just a mutable tag:

- `ensure-actionlint.sh` fetches `download-actionlint.bash` from the **immutable commit SHA** of the v1.7.5 tag (not the `v1.7.5` tag), passes the version explicitly so it installs that exact release, then independently re-verifies the extracted binary against a pinned sha256 (enforced on the CI arch, linux/amd64).
- `ensure-bats.sh` verifies the downloaded release tarball against a pinned sha256 before extraction.
- `ensure-yamllint.sh` installs via `pip install --require-hashes -r infrastructure/requirements-yamllint.txt`, so pip refuses any artefact — yamllint or a transitive dep — whose digest isn't listed. Regenerate that file when bumping (see its header).

When bumping any of these, update the version **and** the matching digest/requirements together. These scripts run only in local hooks and the reusable lint lane's pre-commit equivalents — publish logic lives in `acme-skunkworks/shared-workflows` (`reusable-pkg-release.yml`), which keeps a compromised upstream away from the publish identity.

Configuration: `.yamllint.yml` at the repo root extends defaults, demotes line-length / indentation to warnings (Prettier owns formatting), allows the GitHub Actions truthy values (`on`, `off`, `yes`, `no`), and ignores `node_modules/`, `dist/`, `.turbo/`, `pnpm-lock.yaml`. CI YAML/workflow linting is owned by the shared `reusable-lint.yml` lane (centralised `.yamllint.yml` + actionlint 1.7.12 in shared-workflows); bats/shellcheck run in the shared `reusable-build-test.yml` lane.

Enforcement: pre-commit is best-effort (skip with install hint when missing); CI is the shared `lint` reusable caller in `ci.yml`, always enforced. The local install-and-run logic for both tools lives in `infrastructure/scripts/ensure-yamllint.sh` and `ensure-actionlint.sh` (kept as tested local tooling, no longer invoked directly by `ci.yml`).

## Validating workflows locally with `act`

`actionlint` and `yamllint` catch schema and expression-level mistakes. They say nothing about whether a workflow actually _works_ end-to-end — Node/pnpm setup ordering, env propagation, conditional skips, step interdependencies. [`act`](https://github.com/nektos/act) closes that gap by running the workflow against your local Docker daemon so you can iterate without push-and-pray.

**Install:** `brew install act` (macOS) or `bash <(curl -fsSL https://raw.githubusercontent.com/nektos/act/master/install.sh)` (Linux). Requires a running container engine — Docker Desktop, Colima, or podman. `pnpm act:list` is the smoke test: if it enumerates the jobs in `.github/workflows/`, you're set up.

**`.actrc`** at the repo root pins `ubuntu-latest` to `catthehacker/ubuntu:act-latest` (Ubuntu 24.04-based, matching real `ubuntu-latest`). The default `act` image is intentionally minimal and silently breaks Node/pnpm setups, so don't remove this. Container architecture is deliberately **not** pinned — `act` defaults to the host arch (arm64 on Apple Silicon), which is fast and matches GHA's _results_ for this codebase even though GHA runners are amd64.

**Commands:**

```bash
pnpm act:list           # smoke test — enumerate every job in .github/workflows/
pnpm act:ci             # run ci.yml as a PR event, using .github/act-events/pull_request.json
pnpm act:release:dry    # run pkg-release.yml — stops at OIDC-bound publish steps without a real issuer
```

The PR event fixture lives at `.github/act-events/pull_request.json` and sets `pull_request.head.ref` / `pull_request.base.ref` / `pull_request.title` so the `📓 Changelog completeness` gate (which diffs against `origin/${{ github.base_ref }}` and reads the PR title) resolves real values instead of `origin/`. PR-title lint runs in `validate-pr-title.yml`.

**Capability matrix** for this repo's workflows (the build-once split is validated by static lint — actionlint/yamllint/shellcheck/bats green; the reusable-caller jobs are resolved and exercised in real CI, not locally under `act`, which can't fully run cross-repo reusable-workflow `uses:` callers):

| Workflow / Job                          | Under `act` | Notes                                                                                                                                                                                                                                             |
| --------------------------------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ci.yml` → `config`                     | ✅ full     | Local job — `./.github/actions/load-repo-config` resolves the shared repo-config knobs.                                                                                                                                                           |
| `ci.yml` → `lint` / `build-test`        | ⚠️ partial  | Reusable-workflow callers need cross-repo fetch; `act` may not resolve `acme-skunkworks/shared-workflows@<sha>` without extra setup. Local validation: `pnpm lint:workflows` + push to GHA.                                                       |
| `ci.yml` → `changelog-completeness`     | ✅ full     | On a `feat`/`fix`/breaking title with no `changelog/` entry it fails by design. PR-title lint is in `validate-pr-title.yml`.                                                                                                                      |
| `ci.yml` → `go-no-go`                   | ⚠️ partial  | Aggregator (`GO/NO GO`, `if: always()`) — depends on the reusable-caller job results, so a full verdict only resolves in real CI. This is the required status check.                                                                              |
| `pkg-release.yml` → `pkg-release`       | ⚠️ partial  | Thin caller into `reusable-pkg-release.yml`. Fails at OIDC/provenance steps without a real `ACTIONS_ID_TOKEN_REQUEST_URL` — documented gap. The `npm-release` environment's branch policy is server-side, so `act` ignores it.                    |
| `claude-code-review.yml` / `claude.yml` | ⏭️ skip     | Inline (SHA-pinned) Claude workflows — not reusable callers, which `startup_failure` for the Claude pair (A-621). Need `CLAUDE_CODE_OAUTH_TOKEN`. The `act:*` scripts use `-W` to scope to specific workflows, so these aren't loaded by default. |

**Pre-push gate:** `.husky/pre-push` runs `pnpm lint:workflows` (actionlint) and `pnpm lint:yaml` (yamllint) on every push as a last-line safety net for cases where pre-commit was bypassed. Both are sub-second on this repo. If either tool isn't installed locally the hook prints an install hint and skips — CI is the enforced gate. To bypass entirely in an emergency: `git push --no-verify`.

## `infrastructure/`

`act` validates workflow _wiring_ — that the YAML resolves, steps fire in order, env propagates. It says nothing about whether the logic _inside_ a `run:` block is correct. `infrastructure/` is the home for that logic: shell + TS extracted from workflow `run:` blocks, runnable and unit-tested in isolation. The full conventions document is `infrastructure/README.md`; the high-level rules:

- **Per-script language.** Shell + bats for CLI orchestration (`git`, `gh`, `jq`, `curl`, `pip`). TypeScript + vitest for parsing, branching, anything touching octokit. If a shell script grows past ~20 lines with conditionals, port to TS.
- **Inputs via env, not argv.** Workflows pass values through `env:`; tests mock by passing an env object. No shell quoting drama; clean test seam.
- **Pure functions exported for tests.** Each TS script exports the pure logic; `main()` wires it to real subprocesses. Tests inject a fake runner that records argv.
- **Idempotent.** Re-running with the same inputs is safe. The CI cache-hit branch of `ensure-yamllint.sh` / `ensure-actionlint.sh` / `ensure-bats.sh` is exactly this scenario.
- **Pinned versions in env defaults**, e.g. `ACTIONLINT_VERSION="${ACTIONLINT_VERSION:-1.7.5}"`. The workflow's cache-key still hard-codes the version separately — match them when bumping.

Today's scripts:

| File                                      | Replaces                                                               | Tests                                        |
| ----------------------------------------- | ---------------------------------------------------------------------- | -------------------------------------------- |
| `scripts/ensure-yamllint.sh`              | local yamllint install (CI run now via shared `lint` caller)           | `tests/ensure-yamllint.bats`                 |
| `scripts/ensure-actionlint.sh`            | local actionlint install (CI run now via shared `lint` caller)         | `tests/ensure-actionlint.bats`               |
| `scripts/ensure-bats.sh`                  | local bats install (CI run now via shared `build-test` caller)         | `tests/ensure-bats.bats`                     |
| `send-it/derive-changeset.ts`             | (used by `/send-it`)                                                   | `tests/derive-changeset.test.ts`             |
| `scripts/validate-changelog.ts`           | `validate:changelog` (CI: shared `lint` caller)                        | `tests/validate-changelog.test.ts`           |
| `scripts/check-changelog-completeness.ts` | `ci.yml` `changelog-completeness` gate (A-379)                         | `tests/check-changelog-completeness.test.ts` |
| `scripts/finalise-changelog.ts`           | orchestrator step, run after `release-please release-pr` (A-379/A-376) | `tests/finalise-changelog.test.ts`           |
| `scripts/enrich-changelog.ts`             | (pure lib used by finalise)                                            | `tests/enrich-changelog.test.ts`             |
| `scripts/add-links-changelog.ts`          | (pure lib used by finalise)                                            | `tests/add-links-changelog.test.ts`          |
| `scripts/stamp-changelog-version.ts`      | (pure lib used by finalise)                                            | `tests/stamp-changelog-version.test.ts`      |

CI: ShellCheck, Vitest and bats run in the shared `build-test` reusable caller (`reusable-build-test.yml`) and `validate:changelog` in the shared `lint` caller (`reusable-lint.yml`) — the old local `infra` job is gone. Locally, `pnpm lint:sh` / `pnpm test:sh` skip with install hints if `shellcheck` / `bats` aren't on PATH — `pnpm test` (vitest) always runs because vitest is a node devDep. The TS is typechecked with `pnpm tsc` (`tsc --noEmit`); `tsconfig.json` includes only `infrastructure/**/*.ts` and emits nothing — the published artifact stays `.markdownlint.jsonc`.

When adding workflow-extracted tooling, write the test first, then wire from YAML as a one-liner: `run: pnpm tsx infrastructure/scripts/<name>.ts` or `run: bash infrastructure/scripts/<name>.sh`. (The bespoke `/send-it` slash command and its `infrastructure/send-it/` helpers have been superseded by the shared `send-it` agent skill — see [Agent skills](#agent-skills); the now-orphaned `infrastructure/send-it/` helpers are slated for follow-up removal.)

## Dated changelog (`changelog/`)

The `changelog/` directory is the **only** changelog in the repo — there is no root `CHANGELOG.md` (release-please runs with `skip-changelog`, A-379). It keeps **one dated Markdown file per shippable change** — a browsable, per-change, machine-readable record. `pkg-release.yml` (via `reusable-pkg-release.yml`) sources its GitHub-release notes from these entries. Full schema and lifecycle in **`changelog/README.md`**.

Two-stage lifecycle — finalisation rides inside the release-please release PR, which the private release-orchestrator creates (A-320 / A-376).

1. **PR-time** — `/send-it` Step 5b writes `changelog/<YYYYMMDD-HHMMSS>-<slug>.md` with the PR-time fields (and empty enrichment placeholders), **gated on shippability** (only for shippable changes per Step 5.3 — the same changes that get a release-triggering `feat`/`fix`/breaking PR title), so every entry maps to a version bump. CI's changelog-completeness gate re-enforces this coupling. The entry merges to `main` with its feature PR and sits with placeholders until release.
2. **Release (in the release PR)** — the **orchestrator** runs `release-please release-pr` (which bumps `package.json` + `.release-please-manifest.json`) then `finalise-changelog.ts` (= `pnpm changelog:finalise`). For every entry without a `version`, finalise resolves its merged PR from the `branch` field via `gh` (filling `merged_at`/`commit`/`pr`/`merge_strategy`/`stats`), stamps the just-bumped `version`, and rewrites Linear IDs to links. The orchestrator commits those changelog edits **into the release PR** (pushed with its App token) — so they merge and publish through the normal flow. Idempotent and re-run-safe.

`validate:changelog` enforces the schema (CI: the shared `reusable-lint.yml` caller). Required frontmatter is relaxed to `title`/`created_at`/`category`/`breaking` so backfilled historical entries and in-flight entries both pass.

`finalise-changelog.ts` is the only CLI; `enrich-changelog.ts`, `add-links-changelog.ts`, and `stamp-changelog-version.ts` are pure library modules it composes.

## Release workflow

There are two release modes — know which one you're in.

### Day-to-day releases (CI via OIDC)

Once the package's Trusted Publisher is configured against this repo's `pkg-release.yml`, every release flows through CI:

1. Make changes on a feature branch; `/send-it` bundles, writes the dated `changelog/<slug>.md` entry (for shippable changes), sets a **Conventional Commits PR title** (the squash subject release-please reads — `feat`/`fix`/`feat!` for shippable, a non-release type otherwise), pushes, opens a PR. CI (`ci.yml` + `validate-pr-title.yml`) runs shared lint/build-test callers, the conventional-PR-title lint, the changelog-completeness gate, and the `GO/NO GO` aggregator on the PR.
2. After merge, the private **release-orchestrator** (road-runner-bot, runs a 15-min cron) mints a short-lived repo-scoped App token, runs `release-please release-pr` (which infers the bump from the merged PR titles and writes `package.json` + `.release-please-manifest.json`) then `finalise-changelog.ts`, pushes the `release-please--branches--main` branch, and opens the "`chore(main): release <version>`" release PR. On a later tick it squash-merges that PR once `GO/NO GO` is green.
3. The orchestrator's App-token merge pushes to `main`, re-firing `pkg-release.yml`. The thin caller invokes `acme-skunkworks/shared-workflows/.github/workflows/reusable-pkg-release.yml@<sha>`: an unprivileged build leg `npm pack`s the tarball once (no compile — the artifact is `.markdownlint.jsonc`); the publish leg sees a **freshly bumped version with no matching git tag**, downloads that exact tarball, and publishes it to npm via OIDC Trusted Publishing (no token, no OTP) + provenance attestation, plus git tags + a GitHub release (notes sourced from the dated `changelog/` entry). The GitHub Packages leg downloads the **same** tarball and mirrors it to GitHub Packages with a GitHub-native build-provenance attestation (A-323).

**`pkg-release.yml` is publish-only (A-320).** It does **not** create the release PR — that path needs an identity that isn't `github-actions[bot]` (the "Allow GitHub Actions to create and approve pull requests" toggle is deliberately off, A-313), so versioning lives in the orchestrator where the App key stays private (A-312). A version-vs-tag gate inside `reusable-pkg-release.yml` gates the publish: a feature-merge (version unchanged → its tag exists) is a clean green no-op; a release-PR merge (version bumped → no tag yet) publishes. This is a keyless replacement for the old `.changeset/*.md` scan — no Changesets dependency. The bot's private key never touches this public repo's CI.

**Cross-boundary hardening (A-326).** npm Trusted Publishing binds its OIDC subject to repository + **caller** workflow filename (`pkg-release.yml`, A-543) + environment — not the trigger event, ref, actor, or the reusable callee — so anything able to run `pkg-release.yml` against an arbitrary ref could mint a valid publish credential. Three layers close that:

- **No `workflow_dispatch`.** The only trigger is `push: [main]`; re-run a failed release via "Re-run jobs" on the original push run. The caller stub owns the trigger — `workflow_call` cannot, and a dispatched run would satisfy the same npm OIDC subject as a legitimate post-merge push (A-326).
- **Branch-restricted `npm-release` environment** on the reusable publish jobs. It permits deployments **only from `refs/heads/main`**, so a non-main ref is rejected before the OIDC token is mintable. **No required reviewers** — releases stay hands-off; this is a structural ref gate, not a manual approval. A reusable workflow's `environment:` resolves in the **caller** repo, so it is configured here in repo settings (not in YAML): `gh api -X PUT repos/acme-skunkworks/markdownlint-config/environments/npm-release` with `deployment_branch_policy.custom_branch_policies=true`, then a single `main` branch policy.
- **Version-vs-tag gate** inside `reusable-pkg-release.yml`: a feature-merge (version unchanged → its tag exists) is a clean green no-op; a release-PR merge (version bumped → no tag yet) publishes.

**Build once, publish the exact artifact (A-328).** Implemented in `reusable-pkg-release.yml`: pack-time code (`pnpm install` + `npm pack`) runs **only** in an unprivileged build leg (`contents: read`, no `id-token`/`packages`/`contents: write`). Both publish legs download and ship that one tarball, so a compromised install-time dependency never runs alongside a mintable publish credential, and the npm tarball, the GitHub Packages tarball, and the attested digest are guaranteed byte-identical.

**The publish internals are centralised in `reusable-pkg-release.yml` (A-384); this repo no longer ships local `publish-*.sh` wrappers.** The npm-upgrade step, the raw-npm publish wrapper (`pnpm publish` doesn't satisfy npm Trusted Publishing's OIDC, so it calls `npm publish "$TARBALL" --access public --provenance` directly on the prebuilt tarball), the build-once split, the GitHub Packages leg, and the version-vs-tag gate all live in the reusable workflow now — the same logic the old inline `release.yml` + `publish-via-raw-npm.sh` / `publish-to-github-packages.sh` scripts carried, moved upstream to kill the copy-paste drift. Key properties the reusable workflow preserves:

- **npm upgrade for Trusted Publishing.** `actions/setup-node` prepends its tool-cache bin to PATH, so plain `npm` resolves to whatever Node 22 ships; npm Trusted Publishing requires npm 11.5.1+. The reusable workflow pins the upgrade (not `@latest`) for CI reproducibility and fixes up `$GITHUB_PATH`.
- **Idempotent publishes.** Each leg skips (exits 0) if `npm view name@version` already succeeds instead of re-publishing.
- **GitHub Packages secondary target (A-323).** npmjs.org (OIDC + provenance) is the canonical public source; GitHub Packages is mirrored alongside it. `packages: write` is scoped to its own leg — never the leg that holds `id-token: write` for npm OIDC. Auth is the ephemeral per-job `GITHUB_TOKEN` (no OIDC Trusted-Publisher flow exists for GitHub Packages); provenance is a GitHub-native `actions/attest-build-provenance` attestation over the exact tarball (`gh attestation verify <tarball> --repo acme-skunkworks/markdownlint-config`). The GitHub Packages target is hard-coded to `https://npm.pkg.github.com` and aborts if it drifts (A-330) — the ephemeral `GITHUB_TOKEN` is a bearer credential, so the host must never be redirectable by a config edit.

> **Watch-item:** the npm leg's git tag + GitHub release are created explicitly by the reusable workflow (npm publish itself creates neither). Confirm that step still runs on each release, and that its notes resolve from the dated `changelog/` entry for the version (there is no root `CHANGELOG.md` to fall back on).

Don't reintroduce `NPM_TOKEN` **as a CI secret** unless OIDC is verified broken. The local `.env`-based `NPM_TOKEN` is a different concern — it's for laptop-driven publishes only, never CI.

**Choosing the bump.** There is no changeset file. The bump is inferred by release-please from the **Conventional Commits PR title** (the squash subject): `feat:` → minor, `fix:`/`perf:`/`revert:` → patch, a `!` breaking marker (or `BREAKING CHANGE:` footer) → major. `/send-it` maps shippable changes to `feat`/`fix`/`feat!` only; release-please still bumps on a manually titled `perf:` or `revert:`. Non-release types (`docs:`/`chore:`/`ci:`/`refactor:`/`test:`/`build:`/`style:`) don't cut a release. The conventional-PR-title lint (`validate-pr-title.yml`) + the changelog-completeness gate in `ci.yml` keep the title honest.

### Out-of-band repo settings (one-time, not in code)

These can't live in the repo and must be set in GitHub/npm settings for the day-to-day flow to work (mirror A-348):

- The **`npm-release` environment** restricted to `main` (see the `gh api` command above).
- **`GO/NO GO`** as a required status check on `main` (the `go-no-go` aggregator replaces the old `🔬 Build & Lint` check); **`pr-title / Validate PR title is a Conventional Commit`** is required too, per the estate ruleset.
- **road-runner-bot** installed on this repo (done 2026-06-23 via release-orchestrator PR #11) and this repo listed in the orchestrator's `matrix.repo`.
- The **npm Trusted Publisher** mapping (configured after the first publish — see Bootstrap below).

### Manual publish (break-glass — CI-down only, after the package exists)

> **This is break-glass, not a routine path (A-331).** Reach for it only when CI/OIDC is genuinely down — every normal release goes through `pkg-release.yml` (OIDC, no standing token). The `.env` `NPM_TOKEN` is a long-lived credential: store it in a secrets manager retrieved just-in-time, keep its lifetime short with a documented rotation cadence, rotate immediately on exposure, and never add it as a CI secret.

Auth setup (one-time, or after rotating your token):

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

> The predecessor `@robeasthope/markdownlint-config` was published from `RobEasthope/protomolecule` and continues to exist on npm under that old name. At the new `@acme-skunkworks` scope this is a **fresh package**, so npm's first-publish 2FA enforcement applies. Follow the sequence below for the first publish from this repo.

The very first publish of a brand-new npm package **cannot go through CI**. Two reasons that compound:

- npm (unlike PyPI) has no pending-Trusted-Publisher flow. The package must exist on the registry before the Trusted Publisher form is reachable at `npmjs.com/package/<name>/access`.
- npm enforces interactive 2FA at the publish endpoint for the first publish of a new package, irrespective of account/org/token bypass settings — Granular bypass-2FA tokens only honour the bypass on _subsequent_ publishes. So a non-interactive CI token can't clear it; the first publish has to be done by a human at a terminal with a browser.

So bootstrap is always: manual first publish → configure Trusted Publisher → CI takes over from publish #2.

**Pre-flight:**

- You belong to the target npm org with publish rights.
- npm CLI ≥ 11.5.1 (`npm install -g npm@latest`). A recent npm (11.12.1 verified) does the browser 2FA flow below; older npm or a non-interactive host falls back to the recovery-code path.
- An **interactive browser** is available and npm is on its default `auth-type=web` (don't override it to `legacy`).
- Account has 2FA enabled, with a **passkey registered** (preferred — it completes first-publish 2FA in the browser). **Recovery codes generated and saved** as a fallback.
- `package.json` (and `.release-please-manifest.json`) is at the version you want to ship. For a brand-new package, set it by hand; release-please takes over bumping from publish #2 once the manifest is seeded.

**Sequence:**

1. Set `package.json` and `.release-please-manifest.json` to the version you want to ship — edit directly; no release-please run is needed for the very first publish.
2. `pnpm run release:manual:dry` — verify tarball + auth. **Note:** dry-run does NOT trigger 2FA enforcement, so a successful dry-run does not predict a successful real publish. It only proves the tarball is valid and your token authenticates.
3. `pnpm run release:manual` (or `npm publish --access public --provenance=false`) — **the primary path.** On a recent npm with `auth-type=web`, this **opens a browser and prompts for a passkey/WebAuthn approval** (Touch ID / Face ID / security key). Approve it and the brand-new package publishes cleanly — **no `--otp` needed.**
4. **Recovery-code fallback** — only when the browser flow isn't available (headless host, no passkey registered, or an npm too old for web auth). There the publish fails with `EOTP`; re-run passing a **recovery code as the `--otp` value**:

   ```bash
   npm publish --access public --provenance=false --otp=<recovery-code>
   ```

   Generate codes at npmjs.com → Profile → Two-Factor Authentication → Manage Recovery Codes. Each is single-use (a long hex string, not a 6-digit TOTP — npm accepts it as `--otp` anyway). After the publish succeeds, **immediately regenerate recovery codes**: the one you used is burnt, and if you transmitted it anywhere treat the rest of the set as compromised. (Moot when the passkey browser flow worked — no OTP was ever entered.)

5. Configure Trusted Publisher: `https://www.npmjs.com/package/@acme-skunkworks/markdownlint-config/access` → GitHub Actions → org `acme-skunkworks`, repo `markdownlint-config`, workflow filename (`pkg-release.yml` — the caller stub, not the reusable callee, which is what npm's OIDC subject matches), environment `npm-release`.
6. From here on, releases go through CI cleanly.

### Things that look like solutions but aren't

Saving these to spare the next bootstrap from rediscovering them:

- Toggling "Require 2FA for write actions" off in account settings.
- Disabling org-level 2FA enforcement.
- Generating a Granular token with bypass-2FA enabled — works for publish #2+, NOT publish #1.
- `oathtool` for generating TOTP — only works if you have a TOTP secret, and **npm has phased TOTP out of new accounts** (passkeys / security keys + recovery codes are what's offered now).
- Disabling 2FA entirely — npm's policy _requires_ either 2FA or a bypass-2FA token; you can't disable both. And the bypass token doesn't help for publish #1 anyway.

**The passkey browser flow is the primary answer**, with recovery codes as the fallback for when that flow isn't available. (Earlier copies of this runbook listed `npm publish --auth-type=web` as ignored-by-publish and treated recovery codes as the only OTP-shaped value a passkey account can produce — that was true on older npm, but `auth-type=web` is now the default and is exactly what triggers the browser passkey approval that clears first-publish 2FA.)

## OIDC Trusted Publisher

The Trusted Publisher mapping on npm is what makes the OIDC publish work without an `NPM_TOKEN`. It is configured **after** the manual bootstrap publish (the npmjs.com form is unreachable until the package exists):

- After the first version lands on npm via the manual bootstrap, configure the Trusted Publisher: `npmjs.com/package/@acme-skunkworks/markdownlint-config/access` → GitHub Actions → org `acme-skunkworks`, repo `markdownlint-config`, workflow `pkg-release.yml` (the caller stub, which is what npm's OIDC subject matches — not the reusable callee), environment `npm-release`.
- From the next release onward, releases ride CI/OIDC with provenance attestation — no token, no OTP, no recovery codes.
- The legacy `@robeasthope/markdownlint-config` on npm has its own independent Trusted Publisher mapping pointing at `RobEasthope/protomolecule` — a different package; leave it alone.
