# security-standards

Team security standards for developers and AI agents: application security, identity, CORS, secrets, threat modeling, CI/CD supply chain, dependencies and SBOM.

Part of the Slight76 standards handbooks indexed at [standards-marketplace](https://github.com/Slight76/standards-marketplace). Written for a small team and its AI agents.

## Documents

| Document | Covers | Rule prefixes |
| --- | --- | --- |
| [docs/security-architecture.md](docs/security-architecture.md) | Security design baseline: authentication and authorization on every operation, threat documentation, browser credential rules, sensitive data lifecycle | SEC-001..004 |
| [docs/identity-standard.md](docs/identity-standard.md) | Identity profiles, token audience validation, resource/tenant authorization, browser sessions, CSRF, no custom protocols | IAM |
| [docs/cors-standard.md](docs/cors-standard.md) | Same-origin vs cross-origin decision, exact per-environment origin allow-lists, credentials, ASP.NET Core fragment, verification matrix | CORS |
| [docs/application-security-standard.md](docs/application-security-standard.md) | Threat model scope, input validation, SSRF, uploads, rate limits, CSP, secrets and telemetry, webhooks, negative test matrix | SEC-005..007 |
| [docs/secrets-management.md](docs/secrets-management.md) | Where secrets live (user-secrets, .env, Fly.io, GitHub environments), nothing in git/images/logs, push protection, rotation, leak response | SECR |
| [docs/threat-modeling-guide.md](docs/threat-modeling-guide.md) | Lightweight STRIDE-per-boundary process, triggers, filling the template, mitigations mapped to rule IDs | TM |
| [docs/ci-cd-and-supply-chain-security.md](docs/ci-cd-and-supply-chain-security.md) | SHA-pinned actions, least-privilege permissions, OIDC and short-lived tokens, protected environments, trigger footguns, provenance attestations | SCS |
| [docs/dependency-and-sbom-policy.md](docs/dependency-and-sbom-policy.md) | Lockfiles, Dependabot/Renovate cadence, vulnerability scanning, severity SLAs, CycloneDX SBOM per release, license allow-list | DEP |
| [templates/threat-model.md](templates/threat-model.md) | Threat model table template used by the guide | (template) |

## Read by task

See [skills/security-standards/SKILL.md](skills/security-standards/SKILL.md).

## Install as an agent skill

| Agent | Command |
| --- | --- |
| Copilot CLI | `copilot plugin marketplace add Slight76/standards-marketplace` then `copilot plugin install security-standards@slight76-standards` |
| GitHub CLI (any agent) | `gh skill install Slight76/security-standards security-standards --scope user --pin v1.0.0` |
| Claude Code | `/plugin marketplace add Slight76/standards-marketplace` then `/plugin install security-standards@slight76-standards` |

## Layout

| Path | Purpose |
| --- | --- |
| `docs/` | Standards documents (frontmatter, applies-when, rule table) |
| `catalog/catalog.json` | Machine-readable rules; `externalDecisions` points at historic ADRs |
| `adr/` | Decisions local to this handbook |
| `skills/security-standards/` | Agent skill and references |
| `templates/` | Templates specific to this domain (shared ones live in the marketplace) |

Validation: `py ../standards-marketplace/tooling/validate.py --root .`. License: [MIT](LICENSE).
