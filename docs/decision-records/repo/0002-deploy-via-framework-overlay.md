# ADR-repo/0002: Deploy via Framework Overlay

| Field | Value |
| --- | --- |
| Status | Accepted |
| Date | 2026-06-02 |
| Authors | Nick Warila (@NWarila) |
| Decision-maker | Nick Warila (sole portfolio maintainer) |
| Consulted | github-terraform-framework reusable deploy workflow. |
| Informed | Maintainers reviewing deploy topology changes in this runner. |
| Reversibility | Low |
| Review-by | N/A (Accepted) |

## TL;DR

`github-terraform-runner` delegates every `terraform plan` and `terraform apply`
to `nwarila-platform/github-terraform-framework` by calling its reusable deploy
workflow at a pinned SHA. Runner inventory is injected into the framework
workspace via a path overlay at call time; no Terraform module lives in this
repo.

## Context and Problem Statement

The runner owns repository inventory (YAML files) and the framework owns the
Terraform module, providers, and backend configuration. At deploy time the
runner's YAML files must be visible inside the framework's Terraform workspace.
Two approaches were considered:

1. Commit the framework module into this repo so the whole workspace is local.
2. Check out the framework at a pinned ref and overlay the runner's inventory
   on top of that workspace before running Terraform.

## Decision Drivers

- Keep framework implementation in a single source of truth
  (`github-terraform-framework`).
- Allow independent versioning of runner inventory and framework logic.
- Avoid duplicating Terraform module code across multiple runner repos.
- Make the exact framework version auditable via a pinned SHA in
  `terraform-deploy.yaml`.

## Considered Options

1. Embed the framework Terraform module locally in this repo.
2. Check out the framework at a pinned SHA and overlay runner inventory at
   call time.

## Decision Outcome

Chosen option: **Option 2, framework checkout with overlay.**

`terraform-deploy.yaml` calls
`nwarila-platform/github-terraform-framework/.github/workflows/reusable-terraform-deploy.yaml`
at a pinned `framework_ref` SHA. The reusable receives `overlay_paths` that map
runner paths to framework workspace paths:

```
terraform/public => terraform/repos/public
tests/fixtures/terraform/private => terraform/repos/private
```

Private inventory files are fetched from S3 at deploy time and placed into the
same overlay target. The framework then runs `terraform init`, `terraform plan`,
and, on `main` push, `terraform apply`.

The same overlay is used by `pr-validation.yaml` during PR checks, so the plan
seen in CI matches what apply will execute.

## Pros and Cons of the Options

### Option 1: Embed Terraform module locally

- Good, because the full workspace is available without a secondary checkout.
- Bad, because changes to the framework must be manually synced into every
  runner, creating drift risk.
- Bad, because the runner's purpose (inventory deployer) and the framework's
  purpose (module) become conflated.

### Option 2: Framework checkout with overlay (chosen)

- Good, because framework changes flow to all runners on the next ref bump.
- Good, because the runner diff is limited to inventory YAML and overlay config.
- Good, because the pinned SHA makes the exact framework version auditable.
- Bad, because debugging requires opening the framework repository.
- Bad, because overlay misconfiguration can silently omit inventory files; the
  Terraform plan output in PR CI is the verification mechanism.

## Confirmation

This decision is confirmed when:

- `terraform-deploy.yaml` calls the framework reusable by immutable SHA;
- `pr-validation.yaml` uses the same `framework_ref` SHA as `terraform-deploy.yaml`;
- `overlay_paths` maps all runner inventory paths to their framework-relative
  equivalents; and
- no Terraform module (`*.tf` files other than fixtures) is tracked in this repo.

## Consequences

Renovate keeps `framework_ref` current via a configured datasource. Bumping the
ref is a low-risk change when the framework's changelog shows no breaking module
changes. Breaking changes require reviewing the overlay path mapping and any
new required inputs before merging the bump.

## Assumptions

- `nwarila-platform/github-terraform-framework` remains the single Terraform
  module implementation for `NWarila` account repositories.
- The overlay path convention (`terraform/repos/public`, `terraform/repos/private`)
  remains stable in the framework.

## Supersedes

None.

## Superseded by

None.

## Implementing PRs

Introduced by the same PR that adds `docs/diagrams/deploy-overlay-sequence.mmd`
documenting this topology.

## Related ADRs

- [ADR-repo/0001](0001-thin-runner-deployer-scope.md) — thin runner scope that
  motivates this deploy approach.
- [ADR-0005 (org)](../org/0005-pin-terraform-and-provider-versions-exactly.md)
- [ADR-0006 (org)](../org/0006-keep-github-control-planes-namespace-local.md)

## Compliance Notes

This ADR records deploy architecture. No inventory changes result from it.
