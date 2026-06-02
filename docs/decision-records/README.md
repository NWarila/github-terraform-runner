# Architecture Decision Records

This directory holds decision records that are relevant to this runner.

- `docs/decision-records/org/` mirrors org-baseline ADRs from `NWarila/.github`.
- `docs/decision-records/repo/` is reserved for runner-specific ADRs.

Template-tier ADRs from `terraform-runner-template` are maintained in the
template repository. This runner can link to them from repo-specific ADRs when
needed, but it does not carry local copies as enforced runtime surface.

## Repo ADRs

| # | Title | Status | Date |
| --- | --- | --- | --- |
| [repo/0001](repo/0001-thin-runner-deployer-scope.md) | Thin Runner Deployer Scope | Accepted | 2026-06-02 |

## Org-Mirrored Index

| # | Title | Status | Date |
| --- | --- | --- | --- |
| [org/0001](org/0001-use-architecture-decision-records.md) | Use Architecture Decision Records to Document Design Rationale | Accepted | 2026-04-22 |
| [org/0002](org/0002-adopt-diataxis-documentation-framework.md) | Adopt DiÃ¡taxis as the Documentation Framework | Accepted | 2026-04-24 |
| [org/0003](org/0003-use-deny-all-gitignore-strategy.md) | Use a Deny-All `.gitignore` Strategy | Accepted | 2026-04-25 |
| [org/0004](org/0004-use-renovate-for-dependency-updates.md) | Use Renovate for Dependency Updates with Per-Template Baselines | Accepted | 2026-05-05 |
| [org/0005](org/0005-pin-terraform-and-provider-versions-exactly.md) | Pin Terraform and Provider Versions Exactly | Accepted | 2026-05-05 |
| [org/0006](org/0006-keep-github-control-planes-namespace-local.md) | Keep GitHub Control Planes Namespace-Local | Accepted | 2026-06-01 |
| [org/0007](org/0007-centralize-universal-ci-reusables-within-each-namespace.md) | Centralize Universal CI Reusables Within Each Namespace | Accepted | 2026-06-02 |
| [org/0008](org/0008-enforce-repo-hygiene-by-repo-type.md) | Enforce Repo Hygiene by Repo Type | Accepted | 2026-06-02 |
| [org/0009](org/0009-classify-baseline-manifest-byte-identity.md) | Classify Baseline Manifest Byte Identity | Accepted | 2026-06-02 |
| [org/0010](org/0010-keep-ai-attribution-out-of-version-control.md) | Keep AI Attribution Out of Version Control | Accepted | 2026-06-02 |
