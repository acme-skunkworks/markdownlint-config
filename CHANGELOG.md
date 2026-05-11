# @acme-skunkworks/markdownlint-config

## 2.0.0

### Major Changes

- 60c1cbd: **Breaking change for consumers.** Re-enabling MD031/MD034/MD036/MD051, switching MD029 from off to `style: "ordered"`, and adding the new cli2 0.22 rules (MD052/053/056/058) will fire on existing markdown that previously passed. Cutting to `2.0.0` so the strictness increase is opted into via a deliberate bump rather than surprise-shipped to `^1.0.0` pinners on next `pnpm install`.

  Audit and adjust the inherited rule set; convert config to `.markdownlint.jsonc` with inline per-rule rationale (ASW-92).

  **Re-enabled (were off in v1.0, now back on via `"default": true`):**
  - `MD031` — blank lines around fenced code blocks (standard convention; `--fix` resolves).
  - `MD034` — bare URLs (`--fix` wraps in `<>`).
  - `MD036` — emphasis used as heading (almost always a typo for a real heading).
  - `MD051` — link fragments (fragment resolution improved in `markdownlint` 0.40).

  **Newly available via the `markdownlint-cli2` 0.18 → 0.22 bump (`markdownlint` 0.38 → 0.40), all enabled at defaults:**
  - `MD052` — reference links/images use defined labels.
  - `MD053` — unused reference definitions.
  - `MD056` — table column count.
  - `MD058` — blank lines around tables.

  **Adjusted:**
  - `MD029` — re-enabled with `style: "ordered"` (sequential `1. 2. 3.` numbering).

  **Kept off (with rationale now anchored in the config file itself):**
  - `MD013` (line length) — prose concern; editors handle wrapping.
  - `MD033` (inline HTML) — MDX embeds JSX/HTML routinely.
  - `MD040` (fence language) — friction with pseudo-code and CLI output samples; re-disabled in upstream protomolecule PR #345 and confirmed here.
  - `MD041` (first-line H1) — breaks fragment/partial files designed for embedding.

  **Kept as configured:**
  - `MD003` (ATX-style headings), `MD007` (2-space list indent), `MD024` (siblings_only) — all unchanged.

  **Other changes:**
  - `.markdownlint.json` → `.markdownlint.jsonc`; `package.json` `main` and `files` updated accordingly. Consume via `markdownlint-cli2` (its parser stack handles JSONC). Bare `markdownlint` using `JSON.parse` will reject the comments.
  - `markdownlint-cli2` dev-dep bumped `^0.18.1` → `^0.22.1`.
  - README "Configuration Details" rewritten; "Philosophy" section now states the **strict-on-structure, lenient-on-prose** split explicitly.

## 1.0.0

### Major Changes

- c11a08f: Initial release of `@acme-skunkworks/markdownlint-config` — a shared markdownlint configuration for consistent Markdown formatting across projects. Consumers extend `.markdownlint.json` directly via `markdownlint-cli2`.
