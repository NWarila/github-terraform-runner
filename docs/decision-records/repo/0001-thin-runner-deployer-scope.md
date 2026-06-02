# ADR-repo/0001: Thin Runner Deployer Scope

| Field | Value |
| --- | --- |
| Status | Accepted |
| Date | 2026-06-02 |
| Authors | Nick Warila (@NWarila) |
| Decision-maker | Nick Warila (sole portfolio maintainer) |
| Consulted | Terraform runner template contract and repository cleanup findings. |
| Informed | Maintainers reviewing GitHub inventory changes in this runner. |
| Reversibility | Medium |
| Review-by | N/A (Accepted) |

## TL;DR

`NWarila/github-terraform-runner` is a thin data-only deployer for the
`NWarila` GitHub account. It owns repository inventory, caller workflows, deploy
inputs, public-safe private fixtures, and repo-specific documentation. It does
not own a local Terraform module, template-maintainer tools, OPA policies,
contract fixtures, or local reusable workflow implementations.

## Context and Problem Statement

This runner deploys GitHub repository settings by passing local inventory into
`nwarila-platform/github-terraform-framework`. The framework owns the Terraform
module, tests, and apply implementation. `NWarila/terraform-runner-template`
owns the runner contract, validation reusable, and thin caller shape.

Earlier versions of this repo carried copied template surfaces that did not
serve a local job in the runner: helper tools, policy fixtures, and local
reusable workflow implementations. Those files made the repo look like a
framework even though the operational job is narrower: declare repo inventory
and call the framework.

## Decision Drivers

- Keep the repo reviewable as inventory plus caller wiring.
- Avoid dead local copies of template validators and policies.
- Keep framework implementation in `github-terraform-framework`.
- Keep runner contract enforcement in `terraform-runner-template`.
- Make any repo-specific exception explicit through repo ADRs and docs.

## Considered Options

1. Keep this repo as a thin data-only runner deployer.
2. Let this repo mirror the full runner template toolchain locally.
3. Move the GitHub Terraform module into this repo and stop using the framework.

## Decision Outcome

Chosen option: **Option 1, keep this repo as a thin data-only runner deployer.**

This repo owns:

- public inventory under `terraform/public/`;
- private inventory staging under `terraform/private/` and public-safe private
  fixtures under `tests/fixtures/terraform/private/`;
- workflow callers for validation, drift gate, security, repo hygiene, release,
  auto-merge, and deploy, plus the scheduled org-ADR mirror auto-sync caller
  (`org-adr-auto-sync.yaml`, documented in `docs/reference/mirroring.md`);
- deploy-specific secrets, variables, and environment inputs configured in
  GitHub; and
- repo-specific documentation and ADRs.

This repo must not own:

- `tools/`;
- `policies/`;
- `Makefile`;
- local `.github/workflows/reusable-*.yaml` or `.yml` workflow bodies; or
- an executable Terraform module that replaces
  `nwarila-platform/github-terraform-framework`.

## Pros and Cons of the Options

### Option 1: Thin data-only runner deployer (chosen)

- Good, because every local file has a direct job in inventory deployment.
- Good, because validation logic stays with the template and framework that own
  it.
- Good, because contract checks can reject framework-shaped drift.
- Bad, because debugging delegated validation requires opening the template or
  framework repo.
- Bad, because template contract bumps may require caller and docs updates in
  this repo.

### Option 2: Mirror the full template toolchain locally

- Good, because copied validators are available in the checkout.
- Bad, because copied validators can become stale or uninvoked.
- Bad, because the repo becomes harder to distinguish from the template.

### Option 3: Move framework implementation into this repo

- Good, because one repo would contain inventory and implementation.
- Bad, because it breaks the framework/runner split used across the portfolio.
- Bad, because it creates another GitHub Terraform framework to maintain.

## Confirmation

This decision is confirmed when:

- `terraform-runner-template/tools/check_template_contract.py --type runner`
  passes against this repo;
- no local reusable workflow implementation is tracked here;
- `pr-validation.yaml` calls the template validation reusable with
  `mode: runner` and `run_contract_check: true`;
- `terraform-deploy.yaml` calls the framework deploy reusable by immutable ref;
  and
- local documentation describes this repo as an inventory deployer, not a
  framework.

## Consequences

Changes to repository inventory should be small and auditable. Changes to
validation, reusable workflows, policy, or Terraform implementation belong in
the owning template or framework first, followed by a pinned ref bump here.

## Assumptions

- `nwarila-platform/github-terraform-framework` remains the framework that owns
  the GitHub Terraform module.
- `NWarila/terraform-runner-template` remains the runner contract source.
- This repo continues to manage the `NWarila` namespace only.

## Supersedes

None.

## Superseded by

None.

## Implementing PRs

This ADR is introduced by PR #47 (the documentation-governance extraction PR that
also realigns the runner callers to the current thin-runner contract).

## Related ADRs

- [ADR-template/0005](https://github.com/NWarila/terraform-runner-template/blob/main/docs/decision-records/template/0005-enforce-thin-runner-deployer-shape.md)
- [ADR-0008 (org)](../org/0008-enforce-repo-hygiene-by-repo-type.md)
- [ADR-0009 (org)](../org/0009-classify-baseline-manifest-byte-identity.md)

## Compliance Notes

This ADR records repository scope. It does not change which repositories are
managed; inventory changes remain ordinary repo-data PRs under `terraform/`.
