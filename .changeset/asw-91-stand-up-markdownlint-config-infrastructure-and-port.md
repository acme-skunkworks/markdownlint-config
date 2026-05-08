---
"@robeasthope/markdownlint-config": patch
---

Port `@robeasthope/markdownlint-config` to its standalone repo. The published `.markdownlint.json` is byte-identical to v1.1.1 — this release only changes the publish source from `RobEasthope/protomolecule` to `acme-skunkworks/markdownlint-config`, with full standalone-repo infrastructure (CI, husky hooks, `/send-it`, changesets, OIDC-ready `release.yml`).
