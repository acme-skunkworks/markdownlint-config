# @robeasthope/markdownlint-config

## 1.1.1

### Patch Changes

- [`e98d3f7`](https://github.com/RobEasthope/protomolecule/commit/e98d3f7fdaee497ae24a06df75389273d5e0428d) [#345](https://github.com/RobEasthope/protomolecule/pull/345) - Disable MD040 rule (fenced code language requirement)

  The MD040 rule has been causing unnecessary friction by requiring all fenced code blocks to have a language identifier. This change provides more flexibility for markdown authoring while maintaining other code quality standards.

## 1.1.0

### Minor Changes

- [`5c32e87`](https://github.com/RobEasthope/protomolecule/commit/5c32e87c3fb23837780aebb0143c09222176eeec) [#177](https://github.com/RobEasthope/protomolecule/pull/177) - feat(markdownlint-config): add shareable markdownlint configuration package
  - Created new `@robeasthope/markdownlint-config` package for consistent Markdown formatting
  - Extracted configuration from root `.markdownlint-cli2.jsonc`
  - Updated root and all package configs to extend from the new shared package
  - Provides sensible defaults with flexibility for documentation needs

## 1.0.0

### Major Changes

- Initial release of shareable markdownlint configuration
- Extracted configuration from root `.markdownlint-cli2.jsonc`
- Enables consistent Markdown formatting across projects
- Sensible defaults with flexibility for documentation needs
