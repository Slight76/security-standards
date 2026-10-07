---
title: "Dependency and SBOM policy"
status: proposed
version: 1.0.0
owner: "@Slight76"
---
# Dependency and SBOM policy

Baseline: 1.0.0. Applies when: a repository declares third-party packages (NuGet, npm, container base images, GitHub Actions)

Decision: [ADR-0001](../adr/0001-adopt-security-standards.md). Rules become binding when this baseline is adopted; examples explain the policy and do not establish business requirements.

Third-party code is most of what ships. This document defines how a small team keeps it current, knows what is in each release, and fixes vulnerabilities on a predictable schedule. Pipeline hardening is in [CI/CD and supply chain security](ci-cd-and-supply-chain-security.md); adding a dependency still needs the provenance, license, and security review described in [application security](application-security-standard.md) (SEC-005).

## Lockfiles and reproducible restores

| Ecosystem | Lockfile | Restore command in CI |
| --- | --- | --- |
| NuGet | `packages.lock.json` (`<RestorePackagesWithLockFile>true</RestorePackagesWithLockFile>` in `Directory.Build.props`) | `dotnet restore --locked-mode` |
| npm | `package-lock.json` | `npm ci` |
| Docker | Base image pinned by digest (`FROM mcr.microsoft.com/dotnet/aspnet:8.0@sha256:...`) | Build fails if the digest is absent |
| GitHub Actions | Commit SHA in `uses:` (SCS-001) | Workflow lint |

Lockfiles are committed. Floating ranges (`*`, `^` without a lockfile, `latest` tags) are not used in anything that builds in CI. Central Package Management (`Directory.Packages.props`) keeps NuGet versions in one place per repository.

## Update cadence

Dependabot is the default; Renovate is acceptable where a repository needs its grouping or scheduling features. Either way the configuration is committed and reviewed.

```yaml
# .github/dependabot.yml (excerpt)
version: 2
updates:
  - package-ecosystem: nuget
    directory: /
    schedule: { interval: weekly, day: monday }
    groups:
      minor-and-patch: { update-types: [minor, patch] }
  - package-ecosystem: npm
    directory: /web
    schedule: { interval: weekly, day: monday }
    groups:
      minor-and-patch: { update-types: [minor, patch] }
  - package-ecosystem: docker
    directory: /
    schedule: { interval: weekly }
  - package-ecosystem: github-actions
    directory: /
    schedule: { interval: weekly }
```

Weekly grouped PRs for minor and patch updates; individual PRs for majors. Update PRs are merged within the week when CI is green; an update PR older than 30 days is a review-process finding, not a backlog item. Security updates from Dependabot bypass the weekly batch and open immediately.

## Vulnerability scanning in CI

Every build runs both checks and fails on the thresholds below:

```sh
dotnet list package --vulnerable --include-transitive   # fails the job on High/Critical via grep or a script exit code
npm audit --audit-level=high                            # fails on High/Critical
```

GitHub Dependabot alerts stay enabled on every repository and are triaged weekly by the application owner. Container images are scanned at build (`docker scout cves` or Trivy) and the result attached to the release.

## Severity SLAs for fixes

| Severity (CVSS) | Reachable in our code path | Fix deployed within |
| --- | --- | --- |
| Critical (9.0+) | Any | 48 hours |
| High (7.0–8.9) | Yes | 7 days |
| High (7.0–8.9) | No (documented) | 30 days |
| Medium (4.0–6.9) | Any | 30 days, or next scheduled release if sooner |
| Low | Any | Next grouped update |

"Not reachable" must be written down: which API is affected and why the application never calls it. A Dependabot alert dismissed as "not used" with no note is reopened. If no fixed version exists, pin to the least-bad version, add the compensating control to the threat model (TM-002), and record an exception with an expiry.

## SBOM per release

Each release produces a CycloneDX SBOM for every deployable artifact and attaches it to the GitHub release alongside the provenance attestation (SCS-004).

```sh
dotnet tool install --global CycloneDX
dotnet CycloneDX ./src/Api/Api.csproj -o ./artifacts/sbom -j   # api.bom.json
npx @cyclonedx/cyclonedx-npm --output-file ./artifacts/sbom/web.bom.json
```

Container images use `docker sbom` or Syft to include the base image contents. The SBOM is the input for answering "are we affected?" when a new CVE is published; the answer should take minutes, not a code search.

## License allow-list

| Status | Licenses |
| --- | --- |
| Allowed | MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause, ISC, 0BSD, Unlicense, MS-PL, Zlib |
| Review required | MPL-2.0, LGPL-2.1/3.0 (dynamic linking only), CC-BY-4.0 (assets) |
| Not allowed in shipped code | GPL-2.0/3.0, AGPL-3.0, SSPL, BUSL, any "non-commercial" or unlicensed package |

A license check runs from the SBOM (for example `cyclonedx` policy tooling or `license-checker` for npm) and fails on anything outside the allowed set without a recorded review. Dual-licensed packages are recorded with the license actually elected.

## Removing dependencies

Before adding a package ask whether twenty lines of code would do. Review quarterly for unused packages (`dotnet list package --deprecated`, `npx depcheck`) and remove them; every removed package is one fewer thing to patch.

## Rules and required evidence

| ID | Requirement | Verification |
| --- | --- | --- |
| DEP-001 | Repositories MUST commit lockfiles, restore in locked mode in CI, pin container base images by digest, and run automated dependency updates (Dependabot or Renovate) on at least a weekly schedule for every ecosystem they use. | Lockfiles present; CI restore uses locked mode; `dependabot.yml` or `renovate.json` covers nuget, npm, docker, and github-actions as applicable |
| DEP-002 | CI MUST fail on High or Critical vulnerabilities in direct or transitive dependencies, and fixes MUST ship within the severity SLA or carry a documented reachability note or exception. | `dotnet list package --vulnerable` and `npm audit` run in CI; alert triage notes; exception records for overdue items |
| DEP-003 | Every release MUST publish a CycloneDX SBOM per deployable artifact and MUST pass a license check against the allow-list. | SBOM attached to the release; license check job green or reviewed |

## Exceptions

Use the [exception record](https://github.com/Slight76/standards-marketplace/blob/main/templates/exception.md) for a departure. Record affected rules, scope, compensating controls, approval evidence, expiry, and migration path. Agents must not silently replace defaults.
