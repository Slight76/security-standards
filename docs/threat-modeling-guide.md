---
title: "Threat modeling guide"
status: proposed
version: 1.0.0
owner: "@Slight76"
---
# Threat modeling guide

Baseline: 1.0.0. Applies when: a solution adds or changes a trust boundary, persistent store, public contract, identity flow, or external integration

Decision: [ADR-0001](../adr/0001-adopt-security-standards.md). Rules become binding when this baseline is adopted; examples explain the policy and do not establish business requirements.

SEC-002 requires every solution to document threats, trust boundaries, data classification, and mitigation owners. This guide is the lightweight way a small team satisfies it: one session, one table, mitigations mapped to rule IDs. [Threat modeling and secure application behavior](application-security-standard.md) defines the controls; this document defines the process.

## When to run it

| Trigger | Scope of the session |
| --- | --- |
| New solution or service | Full model: every boundary in the data-flow diagram |
| New trust boundary (new client type, public API, webhook, agent tool) | Delta model for the new boundary and anything it touches |
| New persistent store or new data classification | Data lifecycle row set: storage, backup, retention, deletion, telemetry |
| Public contract or identity flow change | Rows for the affected endpoints and session handling |
| Incident or leaked secret | Re-check the affected rows and add the missed misuse case |
| Annual review | Confirm each row still describes the deployed system |

Run the session before implementation starts, in the same PR or ADR that introduces the change. A threat model that arrives after release is an audit, not a design input.

## The process (about an hour)

1. **Draw the boundaries.** A box per component (React SPA, ASP.NET Core API, Postgres, Fly.io edge, GitHub Actions, third-party providers, background workers) and an arrow per data flow. Mark where the trust level changes: browser to edge, edge to API, API to store, API to provider, CI to runtime. A whiteboard photo or a Mermaid diagram in the repository is enough.
2. **Classify the data on each arrow.** Public, internal, personal, or secret. This drives the data lifecycle rows (SEC-004).
3. **Walk STRIDE per boundary.** For each boundary ask the six questions below and write a row only when the answer is a credible misuse.
4. **Pick mitigations from the handbooks, not from imagination.** Each row cites the rule that defines the control (for example IAM-002 for tenant authorization, SEC-007 for webhook replay, SECR-001 for secret exposure, CORS-002 for origin policy). A row with no rule means either the handbooks are missing a control (open an issue) or the mitigation is solution-specific and needs an ADR.
5. **Name a verification and an owner.** A test name, a review step, or a monitoring alert; and the person who owns the residual risk.
6. **Record non-applicability explicitly.** "No file uploads" is a row, so the next reader knows it was considered.

### STRIDE prompts per boundary

| Category | Question to ask at the boundary |
| --- | --- |
| Spoofing | Can the caller pretend to be another user, tenant, service, or provider? (IAM-001, IAM-002, SEC-007) |
| Tampering | Can data in transit or at rest be altered without detection? (SEC-005, SCS-003) |
| Repudiation | Can an action occur without an attributable, retained record? (SEC-004, OBS-001) |
| Information disclosure | Can protected data, secrets, or stack traces leak through responses, logs, or the browser bundle? (SEC-003, SEC-004, SECR-001) |
| Denial of service | Can unbounded input, missing rate limits, or expensive queries exhaust the service? (SEC-005) |
| Elevation of privilege | Can a caller reach an operation or resource beyond their role or ownership? (SEC-001, IAM-002, IAM-003) |

## Filling the template

Copy [templates/threat-model.md](../templates/threat-model.md) to `docs/security/threat-model.md` in the solution repository. Fill the header with solution, revision, scope, owner, review date, and the triggers above. One row per boundary and misuse case:

| Column | What to write |
| --- | --- |
| Boundary/asset | The arrow or store from the diagram, for example "SPA to API: `/api/orders`" |
| Actor and entry point | Who or what sends the input, and through which endpoint, queue, or tool |
| Misuse/failure | The concrete STRIDE scenario in one sentence |
| Mitigation | The control, in enough detail to implement |
| Rule | The rule ID that defines the control |
| Verification | The test, review, or alert that proves the mitigation works, preferably a negative test |
| Residual risk owner | A person, not a team name |

Link the diagram and any accepted exception records from the header. Keep the file under about 150 lines; split by bounded context if it grows.

## Output and evidence

The threat model is complete when every row has a rule ID and a verification, every boundary in the diagram appears at least once, and the PR that introduces the change links the file. `implementation-evidence.json` cites the threat model as the evidence for SEC-002 and TM-001, and the negative tests it names as evidence for the rules in the Rule column.

Review the model in the same code review as the change. Reviewers check that the diagram matches the code, that mitigations are implemented rather than planned, and that each verification exists.

## Rules and required evidence

| ID | Requirement | Verification |
| --- | --- | --- |
| TM-001 | Every solution MUST maintain a threat model from the shared template, with one row per boundary and misuse case, updated in the same change that adds or alters a trust boundary, store, public contract, or identity flow. | Threat model file linked from the introducing PR; diagram matches deployed components |
| TM-002 | Each threat model row MUST map its mitigation to a handbook rule ID (or a solution ADR) and name a verification and a residual risk owner. | Review of the Rule, Verification, and Owner columns; cited tests exist and run in CI |

## Exceptions

Use the [exception record](https://github.com/Slight76/standards-marketplace/blob/main/templates/exception.md) for a departure. Record affected rules, scope, compensating controls, approval evidence, expiry, and migration path. Agents must not silently replace defaults.
