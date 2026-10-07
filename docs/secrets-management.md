---
title: "Secrets management"
status: proposed
version: 1.0.0
owner: "@Slight76"
---
# Secrets management

Baseline: 1.0.0. Applies when: an application, pipeline, or agent uses a credential, key, token, or connection string

Decision: [ADR-0001](../adr/0001-adopt-security-standards.md). Rules become binding when this baseline is adopted; examples explain the policy and do not establish business requirements.

This document narrows SEC-006 (scoped access, rotation, redaction, incident procedures) into concrete practice for a small team running .NET APIs and React SPAs on Fly.io with Postgres, built and deployed by GitHub Actions.

## What counts as a secret

Database connection strings with passwords, API keys, OAuth client secrets, signing keys, webhook signing secrets, Fly.io and cloud deploy tokens, GitHub tokens, SMTP credentials, encryption keys, and private certificates. Public identifiers (OAuth client IDs for public clients, publishable keys, Postgres hostnames) are configuration, not secrets, but still belong in environment-specific configuration rather than source.

Anything a React SPA ships to the browser is public by definition (SEC-003, IAM-004). If a value must stay confidential, it belongs behind the API.

## Where secrets live

| Context | Store | Notes |
| --- | --- | --- |
| Local development | `dotnet user-secrets` for .NET; `.env.local` for Vite | Both are gitignored; never `appsettings.Development.json` with real values |
| Fly.io runtime | `fly secrets set KEY=value -a <app>` | Injected as environment variables; staged secrets deploy on the next release |
| GitHub Actions | Repository or environment secrets | Prefer environment secrets (`production`, `staging`) with required reviewers |
| Postgres | Managed credentials from the provider; one role per application | Rotate by creating a new role/password, switching the secret, then dropping the old one |
| Shared team access | Team password manager | Never chat, email, tickets, or wiki pages |

Read secrets from environment variables through `IConfiguration` (ASP.NET Core binds `ConnectionStrings__Default` to `ConnectionStrings:Default`). Fail fast at startup when a required secret is missing; do not fall back to a hard-coded default.

## Never in git, images, or logs

- Commit no secret-bearing file: `appsettings.*.json` with real values, `.env`, `*.pfx`, `*.pem`, `fly.toml` `[env]` sections with credentials. The `.gitignore` in every repository lists `.env*`, `*.pfx`, `*.pem`, and `**/secrets.json`.
- Docker images: no `COPY .env`, no `ARG` carrying a secret (build args persist in image history). Use BuildKit `--mount=type=secret` if a build needs a private feed token.
- Logs and telemetry: redact `Authorization`, `Cookie`, `Set-Cookie`, `X-Api-Key`, connection strings, and request bodies on identity endpoints. Register a redaction processor once in the logging pipeline rather than trusting each call site. Never log full JWTs; log the `jti` or a hash if correlation is needed.
- Error responses: production uses `UseExceptionHandler`; stack traces and configuration dumps are never returned to clients.
- Agents: AI agents must not paste secrets into prompts, PR descriptions, issues, or generated tests. Tests use fakes or clearly fake values such as `test-key-not-real`.

## Secret scanning and push protection

Enable GitHub secret scanning with push protection on every repository (free for public repositories; enable it in repository settings for private ones). A blocked push is treated as a real finding: remove the value, rotate it, then push. Add a `.gitleaks.toml` or equivalent pre-commit scanner for repositories handling payment, identity, or health data.

Custom patterns are added for internal token formats that GitHub does not recognize, such as webhook signing secrets with a fixed prefix.

## Rotation

| Secret | Trigger | Maximum age |
| --- | --- | --- |
| Deploy and CI tokens | Team membership change, leak, or age | 90 days; prefer OIDC and no stored token at all (see [CI/CD and supply chain](ci-cd-and-supply-chain-security.md)) |
| Database passwords | Leak, role change, or age | 180 days |
| OAuth client secrets, webhook signing secrets | Leak, provider notice, or age | 365 days |
| Signing keys (JWT, data protection) | Leak or algorithm change | Rotate with overlap: publish the new key, accept both, retire the old after the longest token lifetime |

Rotation must be a documented, rehearsed procedure per secret, not a hope. Applications read secrets at startup or through a provider that supports reload; a rotation that requires a code change is a defect.

## When a secret leaks

1. Rotate first, investigate second. Revoke the credential at its issuer within the hour; deploy the replacement.
2. Review access logs for the exposure window (provider audit logs, Postgres `pg_stat_activity` and connection logs, Fly.io logs, GitHub audit log).
3. Purge the value from history only after rotation; a rewritten git history does not un-leak anything, and forks and caches keep copies.
4. Record the incident using the operations handbook's incident process and link it from the threat model (SEC-002, TM-002).
5. Add a scanner pattern or test that would have caught it.

## Rules and required evidence

| ID | Requirement | Verification |
| --- | --- | --- |
| SECR-001 | Secrets MUST NOT be committed to source control, baked into container images, or emitted in logs, traces, or error responses. | Secret scanning with push protection enabled; image history inspection; redaction test on identity endpoints |
| SECR-002 | Runtime and CI secrets MUST live in the platform secret store (Fly.io secrets, GitHub environment secrets) and be read from configuration at startup with fail-fast on absence. | Configuration review; startup test with a missing secret fails |
| SECR-003 | Every secret MUST have a named owner, a rotation procedure, and a maximum age recorded in the application's secrets inventory. | Secrets inventory reviewed at release; rotation rehearsal evidence |
| SECR-004 | A suspected leak MUST trigger revocation and replacement before history cleanup, with the exposure window reviewed and the incident recorded. | Incident record linked from the threat model |

## Exceptions

Use the [exception record](https://github.com/Slight76/standards-marketplace/blob/main/templates/exception.md) for a departure. Record affected rules, scope, compensating controls, approval evidence, expiry, and migration path. Agents must not silently replace defaults.
