# @acme-skunkworks/markdownlint-config

Shared markdownlint configuration for consistent Markdown formatting across projects.

## 📦 Installation

```bash
npm install --save-dev @acme-skunkworks/markdownlint-config
# or
pnpm add -D @acme-skunkworks/markdownlint-config
# or
yarn add -D @acme-skunkworks/markdownlint-config
```

## 🚀 Usage

### Option 1: Direct Extension (Recommended)

Create a `.markdownlint-cli2.jsonc` file in your project root:

```jsonc
{
  "config": {
    "extends": "@acme-skunkworks/markdownlint-config",
  },
  "globs": ["**/*.{md,mdx}", "!**/node_modules/**", "!**/dist/**"],
}
```

### Option 2: Extend in .markdownlint.json

Create or update `.markdownlint.json`:

```json
{
  "extends": "@acme-skunkworks/markdownlint-config"
}
```

> **Note** — Resolving `extends` through this path requires **`markdownlint-cli2`** (its parser stack handles the JSONC comments in the published config). The bare `markdownlint` library or `markdownlint-cli` will try `JSON.parse` on the resolved file and reject the comments. If you're not using cli2, use Option 1 above or copy the rules inline.

### Option 3: Package.json Configuration

Add to your `package.json`:

```json
{
  "markdownlint-cli2": {
    "config": {
      "extends": "@acme-skunkworks/markdownlint-config"
    },
    "globs": ["**/*.{md,mdx}"]
  }
}
```

## 🎯 Philosophy

**Strict on structure, lenient on prose.**

Markdown is two things at once — the structural skeleton of a document (headings, lists, links, tables) and the prose that fills it. This config enforces the first and steps out of the way of the second:

- **Structural rules are enabled.** Headings (including first-line H1 / frontmatter `title:`), list numbering, link references, table shape, blank-line conventions around fences and tables, and fence-language tags — all enforced. These are cheap to get right and the diff noise from violations is high.
- **Prose rules are relaxed.** Line length and inline HTML/JSX are off. These create friction in real-world authoring (long URLs in tables, MDX components) without delivering proportional value.

Each non-default decision in `.markdownlint.jsonc` carries an inline rationale — read it for the "why" before tweaking a rule downstream.

## 📝 Configuration Details

Built on `"default": true` (every markdownlint rule enabled at stock options) with the following non-default decisions:

### Configured rules

- **MD003** — ATX-style headings (`# Heading`), not Setext (`Heading\n===`).
- **MD004** — Unordered-list bullets use `-` only (matches Prettier).
- **MD007** — Nested unordered lists indent by 2 spaces (matches Prettier).
- **MD024** — Repeated heading text allowed across non-siblings (`siblings_only`) — changelogs and per-section API docs need this.
- **MD029** — Ordered lists numbered sequentially (`1. 2. 3.`), not all-ones.
- **MD035** — Thematic breaks are `---` only.
- **MD044** — Proper-name capitalisation (conservative list; `code_blocks` off).
- **MD046** — Code blocks must be fenced.
- **MD048** — Fence style is backtick only.
- **MD049** — Emphasis (italic) uses underscores.
- **MD050** — Strong (bold) uses asterisks.

### Disabled rules

- **MD013** (line length) — prose concern; editors handle wrapping.
- **MD033** (inline HTML) — MDX embeds JSX routinely.

### Notably enabled (formerly disabled)

These were off in v1.0 (carried over from `protomolecule`) and have been re-enabled:

- **MD031** (blank lines around fences), **MD034** (bare URLs), **MD036** (emphasis-as-heading), **MD040** (fence language — use `text` / `console` / `plaintext` for language-less samples), **MD041** (first-line H1, or frontmatter `title:`), **MD051** (link fragments — fragment resolution improved in `markdownlint` 0.40), **MD052/053** (reference link/image hygiene), **MD056/058** (table column count, blank lines around tables).

## 🛠️ Customization

Override specific rules by extending the configuration:

```jsonc
{
  "config": {
    "extends": "@acme-skunkworks/markdownlint-config",
    "MD013": { "line_length": 100 }, // Re-enable line length with a 100-char limit (off in base).
    "MD041": false, // Opt out for repos full of embeddable fragments (on in base).
  },
}
```

## 📋 NPM Scripts

Add these scripts to your `package.json`:

```json
{
  "scripts": {
    "lint:md": "markdownlint-cli2",
    "lint:md:fix": "markdownlint-cli2 --fix"
  }
}
```

Then run:

```bash
# Check Markdown files
pnpm lint:md

# Auto-fix issues
pnpm lint:md:fix
```

## 🔗 Integration

### With Husky and lint-staged

Add to `.lintstagedrc.json`:

```json
{
  "*.{md,mdx}": ["markdownlint-cli2 --fix"]
}
```

### With VS Code

Install the [markdownlint extension](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint) and the configuration will be automatically detected.

## 📖 Rule Reference

For detailed rule documentation, see:

- [markdownlint Rules](https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md)
- [markdownlint CLI2](https://github.com/DavidAnson/markdownlint-cli2)

## 📁 Package Structure

```text
.
├── .markdownlint.jsonc     # Shared configuration (with inline per-rule rationale)
├── package.json
└── README.md
```

## 🤝 Contributing

Found an issue or want to suggest a rule change? Please open an issue in the repository.

## 📄 License

MIT License - see [LICENSE](./LICENSE) file in the root directory.

This software is provided "as is", without warranty of any kind. Use at your own risk.
