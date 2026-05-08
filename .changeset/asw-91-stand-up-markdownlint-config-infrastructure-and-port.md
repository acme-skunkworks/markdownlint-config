---
"@acme-skunkworks/markdownlint-config": major
---

Rename and re-home: the package previously published as `@robeasthope/markdownlint-config@1.1.1` from `RobEasthope/protomolecule` is now `@acme-skunkworks/markdownlint-config`, published from this standalone repo. Consumers must update their dependency name (and any `extends` references inside `.markdownlint-cli2.jsonc`/`.markdownlint.json`) from `@robeasthope/markdownlint-config` to `@acme-skunkworks/markdownlint-config`. The published `.markdownlint.json` configuration is byte-identical to v1.1.1 — no rule changes.
