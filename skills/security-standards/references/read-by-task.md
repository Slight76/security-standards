# Read by task

Full task-to-document map for the security standards handbook. Paths are relative to the repository root. Read the named document(s) and the decision each links; skip the rest.

## Design and architecture

| Task | Read | Key rules |
| --- | --- | --- |
| New solution, service, or trust boundary | `docs/security-architecture.md`, `docs/threat-modeling-guide.md` | SEC-001, SEC-002, TM-001 |
| Data classification, encryption, retention, deletion | `docs/security-architecture.md` | SEC-004 |
| Choosing an identity profile (SPA + API, server-rendered, workload) | `docs/identity-standard.md` | IAM-001 |
| Deciding same-origin vs cross-origin deployment | `docs/cors-standard.md` (decision table) | CORS-001 |
| Threat model for a change; filling the template | `docs/threat-modeling-guide.md`, `templates/threat-model.md` | SEC-002, TM-001, TM-002 |

## Implementation

| Task | Read | Key rules |
| --- | --- | --- |
| Authentication middleware, JWT validation, audiences | `docs/identity-standard.md` | IAM-002 |
| Authorization policies, resource ownership, tenant checks | `docs/identity-standard.md`, `docs/security-architecture.md` | SEC-001, IAM-002 |
| Cookies, sessions, logout, refresh, CSRF/antiforgery | `docs/identity-standard.md` | IAM-001, IAM-003 |
| OAuth/OIDC client in a React SPA; what may live in the bundle | `docs/identity-standard.md`, `docs/security-architecture.md` | SEC-003, IAM-004 |
| Configuring CORS in ASP.NET Core or at the edge | `docs/cors-standard.md` | CORS-002, CORS-003 |
| Input validation, mass assignment, injection | `docs/application-security-standard.md` | SEC-005 |
| URL fetching, SSRF, outbound allow-lists | `docs/application-security-standard.md` | SEC-005 |
| File uploads and downloads | `docs/application-security-standard.md` | SEC-005 |
| Rate limiting, request size limits, CSP, frame policy | `docs/application-security-standard.md` | SEC-005 |
| Receiving webhooks (signatures, replay) | `docs/application-security-standard.md` | SEC-007 |
| Reading configuration and secrets at startup | `docs/secrets-management.md` | SECR-002 |
| Local development secrets (`dotnet user-secrets`, `.env.local`) | `docs/secrets-management.md` | SECR-001, SECR-002 |
| Logging and telemetry redaction | `docs/secrets-management.md`, `docs/application-security-standard.md` | SEC-006, SECR-001 |

## Pipelines and dependencies

| Task | Read | Key rules |
| --- | --- | --- |
| Writing or changing a GitHub Actions workflow | `docs/ci-cd-and-supply-chain-security.md` | SCS-001, SCS-002, SCS-004 |
| Deploying to Fly.io or a cloud provider from CI | `docs/ci-cd-and-supply-chain-security.md`, `docs/secrets-management.md` | SCS-003, SECR-003 |
| Publishing a release, container image, or package | `docs/ci-cd-and-supply-chain-security.md`, `docs/dependency-and-sbom-policy.md` | SCS-004, DEP-003 |
| Adding a NuGet or npm package | `docs/dependency-and-sbom-policy.md`, `docs/application-security-standard.md` | DEP-001, SEC-005 |
| Reviewing Dependabot or Renovate PRs | `docs/dependency-and-sbom-policy.md` | DEP-001 |
| Triaging a CVE or Dependabot alert | `docs/dependency-and-sbom-policy.md` | DEP-002 |
| Generating an SBOM; license check | `docs/dependency-and-sbom-policy.md` | DEP-003 |

## Incidents

| Task | Read | Key rules |
| --- | --- | --- |
| A secret was committed, logged, or pasted somewhere | `docs/secrets-management.md` (When a secret leaks) | SECR-004, SEC-006 |
| Rotating a credential | `docs/secrets-management.md` (Rotation) | SECR-003 |
| Security incident affecting a modeled boundary | `docs/threat-modeling-guide.md`, `docs/application-security-standard.md` | SEC-002, SEC-006 |

## Security review of a PR

Work through the rows that apply and cite rule IDs in the review:

1. Does every new or changed endpoint authenticate and authorize, including resource ownership? (SEC-001, IAM-002)
2. Do cookie-authenticated mutations verify antiforgery? (IAM-003)
3. Did anything confidential land in the browser bundle, a log, a test, or the repository? (SEC-003, SECR-001)
4. Are inputs validated and encoded; are URL fetches, uploads, and rate limits bounded? (SEC-005)
5. Do webhooks verify signatures and reject replays before side effects? (SEC-007)
6. Is CORS exact-origin, credentials-aware, and not used as authentication? (CORS-002, CORS-003)
7. Does the threat model have a row for the new boundary with a rule and a test? (TM-001, TM-002)
8. Are workflow changes SHA-pinned, least-privilege, and free of `pull_request_target` footguns? (SCS-001, SCS-002, SCS-004)
9. Are new dependencies locked, licensed, and vulnerability-free at High/Critical? (DEP-001, DEP-002, DEP-003)
10. Is each negative case covered by a test that demonstrates denial, not just that an option is set?
