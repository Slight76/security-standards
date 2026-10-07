# ADR-0001: Adopt the security standards handbook

Status: Accepted

Date: 2026-10-07

Owner: @Slight76

## Context

The former single `architecture-standards` repository (v0.3.0) mixed every domain into one catalog and skill. Security content (security architecture, application security, identity, CORS, and the threat-model template) was read by few people because it sat beside unrelated backend, data, and platform standards, and it lacked guidance on secrets handling, supply-chain hardening, and dependency hygiene that a small team shipping .NET APIs and React SPAs on Fly.io with GitHub Actions needs every week.

## Decision

Create `security-standards` as the home for security standards, versioned independently from the other handbooks and installable as an agent skill. Documents moved here keep their rule IDs and historical ADR references; new documents are first drafts with `status: proposed`.

## Alternatives

- Keep the domain inside `architecture-standards`: rejected; one repository was too broad to read or install selectively.
- Rewrite all rules from scratch: rejected; existing rules are kept verbatim to preserve consumer baselines.

## Consequences

Consumers pin this repository in `architecture-baseline.json` (`standards[]`). Historic decisions remain in `architecture-standards/adr/` and are declared in `catalog/catalog.json` under `externalDecisions`.

## Traceability

Rule prefixes: SEC, IAM, CORS (moved); SECR, TM, SCS, DEP (new, ADR-0001). Related: standards-marketplace ADR-0001; architecture-standards ADR-0028..0031.

## Verification

`validate.py` passes; `docs.yml` green on `main`; the skill installs through the standards marketplace.

## Approval

@Slight76, 2026-10-07, plan approved in the split planning session.
