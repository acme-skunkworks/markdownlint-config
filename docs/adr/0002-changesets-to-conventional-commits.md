# 0002 — Changesets → Conventional Commits (release-please)

- **Status:** Accepted
- **Date:** 2026-06-23
- **Deciders:** Rob Easthope (sole maintainer of `@acme-skunkworks/markdownlint-config`)
- **Supersedes:** [0001 — Migrate CI from GitHub Actions to CircleCI](./0001-migrate-ci-from-github-actions-to-circleci.md) (the CI question is settled by staying on GitHub Actions)
- **Tracking issues:**
  - [A-379](https://linear.app/acme-skunkworks/issue/A-379) — migrate markdownlint-config to full eslint-config release parity
  - [A-371](https://linear.app/acme-skunkworks/issue/A-371) — switch eslint-config versioning from Changesets to release-please
  - [A-376](https://linear.app/acme-skunkworks/issue/A-376) — adopt release-please in the orchestrator
- **Mirrors:** eslint-config's ADR 0002 (`changesets-to-conventional-commits`), the org-wide pattern this repo adopts.

## Context

`@acme-skunkworks/markdownlint-config` versioned releases with [Changesets](https://github.com/changesets/changesets): contributors wrote a `.changeset/*.md` file declaring the bump, and `changesets/action` opened a "version packages" PR and published on merge. This worked, but the wider `acme-skunkworks` estate (eslint-config, agent-skills, …) is standardising on **release-please** driven by **Conventional Commits**, so that a single private **release-orchestrator** can drive every repo's releases on one path (a `matrix.repo` running `release-please` under `road-runner-bot`).

Keeping this repo on Changesets while its siblings move to release-please would create a **mixed-state estate**: the orchestrator would need two code paths, and a contributor moving between repos would face two different release rituals. Worse, a half-migrated repo (some PRs assuming Changesets, others release-please) has no coherent "what bumps the version?" answer during the window.

## Decision

Adopt **release-please with Conventional Commits**, mirroring eslint-config (A-371) and agent-skills (A-380), in **one atomic PR** (A-379):

- **The PR title is the version signal.** The repo squash-merges, so the squash subject is the PR title; release-please parses it (`feat` → minor, `fix`/`perf`/`revert` → patch, `!`/`BREAKING CHANGE:` → major) to compute the bump. `/send-it` maps shippable changes to `feat`/`fix`/`feat!` only. CI lints the title (`amannn/action-semantic-pull-request`).
- **Versioning is owned by the orchestrator.** release-please runs in the private release-orchestrator (not in this repo's CI), opens the release PR that bumps `package.json` + `.release-please-manifest.json`, and merges it. That merge re-fires `release.yml`, which is **publish-only**, gated by a keyless **version-vs-tag** check (publish iff `package.json`'s version has no matching `v<version>` git tag yet).
- **No root `CHANGELOG.md`.** release-please runs with `skip-changelog: true`. The dated `changelog/<ts>-<slug>.md` subsystem (authored by `/send-it`, finalised by the orchestrator) is the sole changelog and the source of GitHub-release notes.
- **A changelog-completeness CI gate** restores the coupling Changesets gave for free: a release-triggering PR title (`feat`/`fix`/`perf`/`revert`/breaking) must carry a dated `changelog/` entry.
- **GitHub Packages is a secondary publish target**, byte-identical tarball to the npm leg, with a GitHub-native attestation.

The published artifact is unchanged: `.markdownlint.jsonc` itself (no build step, no `dist/`).

## Consequences

**Positive:**

- One release path across the estate; the orchestrator drives every repo identically.
- The bump is derived from the PR title — no separate changeset file to author, lint, or forget.
- `release.yml` holds no version-creation logic and no `RELEASE_PAT`; it is publish-only and keyless (OIDC Trusted Publishing on npm).
- Dual-registry publish (npm + GitHub Packages) from one attested tarball.

**Negative / costs:**

- The PR title is now load-bearing: a mistyped prefix ships the wrong semver. Mitigated by the PR-title lint and the changelog-completeness gate.
- A new TS/tsx/vitest toolchain enters the repo for the `changelog/` tooling (typecheck-only `tsc --noEmit`; nothing is published from it).
- Out-of-band repo settings are required: an `npm-release` environment restricted to `main`, the `🔬 Build & Lint` required check, and `road-runner-bot` installed (done 2026-06-23).

**Reversibility:** git-reversible in-repo. The orchestrator side (A-376) is the coordinated counterpart; both land in the same window.

## References

- eslint-config ADR 0002 (the canonical pattern): <https://github.com/acme-skunkworks/eslint-config/blob/main/docs/adr/0002-changesets-to-conventional-commits.md>
- release-please: <https://github.com/googleapis/release-please>
- Conventional Commits: <https://www.conventionalcommits.org/>
- npm Trusted Publishers: <https://docs.npmjs.com/trusted-publishers/>
