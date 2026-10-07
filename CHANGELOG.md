# Changelog

All notable changes to this handbook. Versions follow SemVer; consumers pin commit SHAs in `architecture-baseline.json`.

## 1.0.0

First release as an independent handbook, split out of `Slight76/architecture-standards` (v0.3.0, the former enterprise-framed repository) at commit `c1bda3d`. See [ADR-0001](adr/0001-adopt-security-standards.md).

### Moved from architecture-standards@c1bda3d (rule IDs unchanged)

- `security/security-architecture.md` -> `docs/security-architecture.md` (SEC-001..SEC-004, ADR-0009)
- `security/identity-standard.md` -> `docs/identity-standard.md` (IAM-001..IAM-004, ADR-0015)
- `security/cors-standard.md` -> `docs/cors-standard.md` (CORS-001..CORS-003, ADR-0014)
- `security/application-security-standard.md` -> `docs/application-security-standard.md` (SEC-005..SEC-007, ADR-0015)
- `templates/threat-model.md` -> `templates/threat-model.md`

Moved documents received frontmatter, team-oriented wording, and absolute cross-handbook links; their rule statements are verbatim.

### New documents (status proposed, ADR-0001)

- `docs/secrets-management.md` (SECR-001..SECR-004)
- `docs/threat-modeling-guide.md` (TM-001..TM-002)
- `docs/ci-cd-and-supply-chain-security.md` (SCS-001..SCS-004)
- `docs/dependency-and-sbom-policy.md` (DEP-001..DEP-003)

### Added

- `catalog/catalog.json` with 27 rules and `externalDecisions` for ADR-0009, ADR-0014, ADR-0015
- Agent skill `skills/security-standards/` with `references/read-by-task.md` and `references/catalog-digest.md`
- `plugin.json` (Agent Plugins 1.0), `AGENTS.md`, `CLAUDE.md`, MIT `LICENSE`, docs lint workflow
