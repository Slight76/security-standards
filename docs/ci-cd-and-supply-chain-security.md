---
title: "CI/CD and supply chain security"
status: proposed
version: 1.0.0
owner: "@Slight76"
---
# CI/CD and supply chain security

Baseline: 1.0.0. Applies when: a repository uses GitHub Actions to build, test, publish, or deploy

Decision: [ADR-0001](../adr/0001-adopt-security-standards.md). Rules become binding when this baseline is adopted; examples explain the policy and do not establish business requirements.

The pipeline is a trust boundary with production credentials, so it gets the same treatment as an API (SEC-001, SEC-002). This document covers the security properties of workflows; pipeline structure, stages, and deployment mechanics live in the operations handbook (CICD-001..CICD-003). Dependency update cadence and SBOM generation are in [dependency and SBOM policy](dependency-and-sbom-policy.md).

## Workflow hardening defaults

```yaml
permissions:
  contents: read          # top-level default; widen per job, never per workflow
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@<40-character-sha> # v4.x, resolve the SHA for the tag you review
        with:
          persist-credentials: false
      - uses: actions/setup-dotnet@<40-character-sha> # v4.x
```

- **Pin every action to a full commit SHA** with the version as a trailing comment. Tags move; SHAs do not. This applies to reusable workflows too (`uses: Slight76/standards-marketplace/.github/workflows/docs-lint.yml@<sha>` at release time).
- **Declare `permissions:` at the top level** as `contents: read` and widen only in the job that needs more (`id-token: write` for OIDC, `packages: write` for publishing, `attestations: write` for provenance). Never rely on the repository default.
- **Disable credential persistence** on checkout unless the job pushes back to the repository.
- **Prefer `ubuntu-latest` hosted runners**; self-hosted runners are not used for public repositories because a fork PR could execute on them.
- **Set a `timeout-minutes`** on every job so a compromised or stuck step cannot hold a runner indefinitely.
- **Minimize `run:` interpolation of untrusted input.** Pass `${{ github.event.* }}` values through `env:` and quote them in the shell; never interpolate issue titles, PR bodies, or branch names directly into a script.

## Authentication to deploy targets

| Target | Required method | Not allowed |
| --- | --- | --- |
| Fly.io | Short-lived deploy token scoped to one app, stored as an environment secret; or `fly tokens create deploy -x 1h` minted in-job where supported | Personal access tokens; organization-wide tokens |
| Cloud providers (AWS, Azure, GCP) | OIDC federation (`id-token: write`) with a role trust policy bound to the repository and environment | Long-lived access keys in secrets |
| GitHub (releases, packages) | The job's `GITHUB_TOKEN` with job-scoped permissions | Classic PATs; fine-grained PATs where `GITHUB_TOKEN` suffices |
| Container registries | OIDC or `GITHUB_TOKEN` for GHCR | Stored registry passwords |

Where a long-lived token is unavoidable, it is an environment secret with a 90-day maximum age (SECR-003) and an owner who rotates it.

## Protected environments

Production deploys run in a GitHub environment named `production` with required reviewers, a deployment branch rule limited to `main`, and production secrets stored only at that environment. Staging mirrors the shape with relaxed approval. A workflow that can deploy to production from an arbitrary branch is a finding regardless of who can trigger it.

Branch protection on `main` requires the `Docs` and build status checks, at least one review, and no force pushes. Dependabot and release automation use the same path; no bypass actors.

## Event triggers

- `pull_request_target` runs with the base repository's secrets against untrusted fork code. Do not use it unless the job performs no checkout of the PR head and runs no code from it (labeling is fine; building is not). Default to `pull_request`, which runs with read-only permissions and no secrets for forks.
- `workflow_run` and `issue_comment` triggers inherit the same risk; treat any payload field as attacker-controlled.
- `workflow_dispatch` inputs are validated with `choice` types where possible and never passed to a shell unquoted.
- Scheduled workflows that publish or deploy are reviewed for what they would do if `main` were compromised.

## Provenance and attestations

Release jobs produce a build provenance attestation for every published artifact (container image, NuGet package, npm package, zip) using `actions/attest-build-provenance` with `attestations: write` and `id-token: write`. Consumers verify with `gh attestation verify <artifact> --owner Slight76` before promotion. Container images are deployed by digest, not by mutable tag, so what was attested is what runs.

Build steps are reproducible from the lockfile: `dotnet restore --locked-mode` and `npm ci`. A restore that changes the lockfile fails the build.

## Keeping actions current

Dependabot is configured for the `github-actions` ecosystem with a weekly schedule and grouped minor/patch updates, so pinned SHAs move only through reviewed PRs. Review Dependabot PRs for actions like any dependency: check the diff of the upstream release, not just the version bump. Remove unused actions rather than leaving them pinned.

```yaml
# .github/dependabot.yml (excerpt)
- package-ecosystem: github-actions
  directory: /
  schedule: { interval: weekly }
  groups:
    actions: { patterns: ["*"] }
```

## Review checklist

Before merging a workflow change confirm: every `uses:` is a 40-character SHA; `permissions:` is declared at the top and widened per job; no `pull_request_target` checks out PR code; production secrets live only in the `production` environment; deploy authentication is OIDC or a short-lived scoped token; publish jobs attest their artifacts; `timeout-minutes` is set.

## Rules and required evidence

| ID | Requirement | Verification |
| --- | --- | --- |
| SCS-001 | Every GitHub Actions `uses:` reference MUST be pinned to a full commit SHA, with Dependabot configured for the `github-actions` ecosystem. | Workflow lint for SHA pins; `dependabot.yml` includes `github-actions` |
| SCS-002 | Workflows MUST declare top-level `permissions: contents: read` and widen permissions only in the jobs that need them. | Workflow review; no job inherits write permissions by default |
| SCS-003 | Deployments MUST authenticate with OIDC federation or short-lived scoped tokens stored as environment secrets, and production deploys MUST run in a protected environment restricted to `main`. | Environment protection settings; absence of long-lived provider keys in repository secrets |
| SCS-004 | Workflows MUST NOT execute code from untrusted pull requests with secrets or write permissions (`pull_request_target` with PR checkout), and release jobs MUST publish build provenance attestations for shipped artifacts. | Trigger review; `gh attestation verify` succeeds for each released artifact |

## Exceptions

Use the [exception record](https://github.com/Slight76/standards-marketplace/blob/main/templates/exception.md) for a departure. Record affected rules, scope, compensating controls, approval evidence, expiry, and migration path. Agents must not silently replace defaults.
