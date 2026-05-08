# @robeasthope/markdownlint-config

Shared markdownlint configuration for consistent Markdown formatting across projects.

## 📦 Installation

```bash
npm install --save-dev @robeasthope/markdownlint-config
# or
pnpm add -D @robeasthope/markdownlint-config
# or
yarn add -D @robeasthope/markdownlint-config
```

## 🚀 Usage

### Option 1: Direct Extension (Recommended)

Create a `.markdownlint-cli2.jsonc` file in your project root:

```jsonc
{
  "config": {
    "extends": "@robeasthope/markdownlint-config",
  },
  "globs": ["**/*.{md,mdx}", "!**/node_modules/**", "!**/dist/**"],
}
```

### Option 2: Extend in .markdownlint.json

Create or update `.markdownlint.json`:

```json
{
  "extends": "@robeasthope/markdownlint-config"
}
```

### Option 3: Package.json Configuration

Add to your `package.json`:

```json
{
  "markdownlint-cli2": {
    "config": {
      "extends": "@robeasthope/markdownlint-config"
    },
    "globs": ["**/*.{md,mdx}"]
  }
}
```

## 📝 Configuration Details

This configuration enforces consistent Markdown formatting with sensible defaults:

### Enabled Rules

- **MD003**: ATX-style headings (`# Heading`)
- **MD007**: Unordered list indentation (2 spaces)
- **Default rules**: All standard markdownlint rules enabled by default

### Disabled Rules

- **MD013**: Line length (disabled for flexibility)
- **MD024**: Multiple headings with same content (only enforced for siblings)
- **MD029**: Ordered list item prefix (flexible numbering)
- **MD031**: Fenced code blocks surrounded by blank lines (disabled)
- **MD033**: Inline HTML (allowed)
- **MD034**: Bare URL detection (disabled)
- **MD036**: Emphasis as heading (disabled)
- **MD040**: Fenced code language tag (disabled for authoring flexibility)
- **MD041**: First line heading requirement (disabled)
- **MD051**: Link fragments (disabled)

## 🛠️ Customization

Override specific rules by extending the configuration:

```jsonc
{
  "config": {
    "extends": "@robeasthope/markdownlint-config",
    "MD013": { "line_length": 100 }, // Enable line length with custom limit
    "MD033": false, // Disable inline HTML
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

- [Markdownlint Rules](https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md)
- [Markdownlint CLI2](https://github.com/DavidAnson/markdownlint-cli2)

## 🎯 Philosophy

This configuration aims to:

1. **Enforce consistency** in heading styles and list formatting
2. **Allow flexibility** in line length and HTML usage for documentation needs
3. **Stay unobtrusive** by disabling overly strict rules

## 📁 Package Structure

```text
.
├── .markdownlint.json      # Shared configuration
├── package.json
└── README.md
```

## 🤝 Contributing

Found an issue or want to suggest a rule change? Please open an issue in the repository.

## 📄 License

MIT License - see [LICENSE](./LICENSE) file in the root directory.

This software is provided "as is", without warranty of any kind. Use at your own risk.
