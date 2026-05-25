# Architecture Decision Records

This directory holds decision records that are relevant to this runner.

- `docs/decision-records/org/` mirrors org-baseline ADRs from `NWarila/.github`.
- `docs/decision-records/repo/` is reserved for runner-specific ADRs.

Template-tier ADRs from `terraform-runner-template` are maintained in the
template repository. This runner can link to them from repo-specific ADRs when
needed, but it does not carry local copies as enforced runtime surface.

## Org-Mirrored Index

| # | Title | Status | Date |
| --- | --- | --- | --- |
| [org/0001](org/0001-use-architecture-decision-records.md) | Use Architecture Decision Records to Document Design Rationale | Accepted | 2026-04-22 |
| [org/0002](org/0002-adopt-diataxis-documentation-framework.md) | Adopt Diátaxis as the Documentation Framework | Accepted | 2026-04-24 |
| [org/0003](org/0003-use-deny-all-gitignore-strategy.md) | Use a Deny-All `.gitignore` Strategy | Accepted | 2026-04-25 |
| [org/0004](org/0004-use-renovate-for-dependency-updates.md) | Use Renovate for Dependency Updates with Per-Template Baselines | Accepted | 2026-05-05 |
| [org/0005](org/0005-pin-terraform-and-provider-versions-exactly.md) | Pin Terraform and Provider Versions Exactly | Accepted | 2026-05-05 |
