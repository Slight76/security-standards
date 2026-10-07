# Catalog digest

Every rule in `catalog/catalog.json` (version 1.0.0) with its statement. Generated from the catalog; regenerate when rules change.

| ID | Statement | Document | Status | Decision |
| --- | --- | --- | --- | --- |
| SEC-001 | Every exposed operation MUST define authentication and server-enforced authorization, including resource ownership. | `docs/security-architecture.md` | Proposed | ADR-0009 |
| SEC-002 | Solutions MUST document threats, trust boundaries, data classification, and mitigation owners. | `docs/security-architecture.md` | Proposed | ADR-0009 |
| SEC-003 | Browser applications MUST NOT contain confidential credentials and token/session handling MUST follow the documented identity design. | `docs/security-architecture.md` | Proposed | ADR-0009 |
| SEC-004 | Sensitive data MUST have defined encryption, retention, deletion, and telemetry redaction controls. | `docs/security-architecture.md` | Proposed | ADR-0009 |
| CORS-001 | Solutions MUST decide whether browser access is same-origin or cross-origin before enabling CORS. | `docs/cors-standard.md` | Proposed | ADR-0014 |
| CORS-002 | Cross-origin policies MUST use validated exact per-environment origins and explicit required capabilities. | `docs/cors-standard.md` | Proposed | ADR-0014 |
| CORS-003 | CORS MUST NOT substitute for authentication, authorization, or CSRF protection. | `docs/cors-standard.md` | Proposed | ADR-0014 |
| IAM-001 | Each solution MUST select and document its browser/workload identity profile and session lifecycle. | `docs/identity-standard.md` | Proposed | ADR-0015 |
| IAM-002 | APIs MUST authenticate tokens for their intended audience and enforce resource/tenant authorization. | `docs/identity-standard.md` | Proposed | ADR-0015 |
| IAM-003 | Cookie-authenticated mutations MUST include independently verified CSRF protection. | `docs/identity-standard.md` | Proposed | ADR-0015 |
| IAM-004 | Agents MUST NOT introduce custom identity protocols or browser client secrets. | `docs/identity-standard.md` | Proposed | ADR-0015 |
| SEC-005 | Exposed boundaries MUST validate inputs and mitigate applicable injection, SSRF, upload, and abuse risks. | `docs/application-security-standard.md` | Proposed | ADR-0015 |
| SEC-006 | Secrets and sensitive telemetry MUST have scoped access, rotation, redaction, and incident procedures. | `docs/application-security-standard.md` | Proposed | ADR-0015 |
| SEC-007 | Webhooks MUST authenticate payloads and prevent unauthorized replay before side effects. | `docs/application-security-standard.md` | Proposed | ADR-0015 |
| SECR-001 | Secrets MUST NOT be committed to source control, baked into container images, or emitted in logs, traces, or error responses. | `docs/secrets-management.md` | Proposed | ADR-0001 |
| SECR-002 | Runtime and CI secrets MUST live in the platform secret store (Fly.io secrets, GitHub environment secrets) and be read from configuration at startup with fail-fast on absence. | `docs/secrets-management.md` | Proposed | ADR-0001 |
| SECR-003 | Every secret MUST have a named owner, a rotation procedure, and a maximum age recorded in the application's secrets inventory. | `docs/secrets-management.md` | Proposed | ADR-0001 |
| SECR-004 | A suspected leak MUST trigger revocation and replacement before history cleanup, with the exposure window reviewed and the incident recorded. | `docs/secrets-management.md` | Proposed | ADR-0001 |
| TM-001 | Every solution MUST maintain a threat model from the shared template, with one row per boundary and misuse case, updated in the same change that adds or alters a trust boundary, store, public contract, or identity flow. | `docs/threat-modeling-guide.md` | Proposed | ADR-0001 |
| TM-002 | Each threat model row MUST map its mitigation to a handbook rule ID (or a solution ADR) and name a verification and a residual risk owner. | `docs/threat-modeling-guide.md` | Proposed | ADR-0001 |
| SCS-001 | Every GitHub Actions `uses:` reference MUST be pinned to a full commit SHA, with Dependabot configured for the `github-actions` ecosystem. | `docs/ci-cd-and-supply-chain-security.md` | Proposed | ADR-0001 |
| SCS-002 | Workflows MUST declare top-level `permissions: contents: read` and widen permissions only in the jobs that need them. | `docs/ci-cd-and-supply-chain-security.md` | Proposed | ADR-0001 |
| SCS-003 | Deployments MUST authenticate with OIDC federation or short-lived scoped tokens stored as environment secrets, and production deploys MUST run in a protected environment restricted to `main`. | `docs/ci-cd-and-supply-chain-security.md` | Proposed | ADR-0001 |
| SCS-004 | Workflows MUST NOT execute code from untrusted pull requests with secrets or write permissions (`pull_request_target` with PR checkout), and release jobs MUST publish build provenance attestations for shipped artifacts. | `docs/ci-cd-and-supply-chain-security.md` | Proposed | ADR-0001 |
| DEP-001 | Repositories MUST commit lockfiles, restore in locked mode in CI, pin container base images by digest, and run automated dependency updates (Dependabot or Renovate) on at least a weekly schedule for every ecosystem they use. | `docs/dependency-and-sbom-policy.md` | Proposed | ADR-0001 |
| DEP-002 | CI MUST fail on High or Critical vulnerabilities in direct or transitive dependencies, and fixes MUST ship within the severity SLA or carry a documented reachability note or exception. | `docs/dependency-and-sbom-policy.md` | Proposed | ADR-0001 |
| DEP-003 | Every release MUST publish a CycloneDX SBOM per deployable artifact and MUST pass a license check against the allow-list. | `docs/dependency-and-sbom-policy.md` | Proposed | ADR-0001 |

Decisions ADR-0009, ADR-0014, and ADR-0015 are historic records in `Slight76/architecture-standards` (declared under `externalDecisions`); ADR-0001 is local to this repository.
