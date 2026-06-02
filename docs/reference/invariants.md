# Invariants

Non-negotiable rules for this runner. Violating one of these is a breaking
change at minimum.

- This repo owns inventory only. It must not contain an executable Terraform
  module for GitHub resources.
- Public repository YAML lives in `terraform/public/`.
- Private repository YAML is fetched into `terraform/private/` at deploy time;
  committed private data is limited to `.gitkeep` and public-safe fixtures.
- Pull request validation must assemble this runner with the pinned
  `nwarila-platform/github-terraform-framework` ref before running framework CI.
- Deploys must use GitHub OIDC for AWS and the fine-grained GitHub token secret
  forwarded through the reusable deploy workflow.

## Template-Family Conventions

- This repository is a runner consumer, not a template or framework. It owns
  inventory, workflow pins, deploy inputs, and runner-specific documentation.
- It does not own `verify.py`, OPA policy, contract fixtures, or template-owned
  reusable workflow implementations. Those stay in `terraform-runner-template`
  and are consumed by immutable SHA.
- It does not own local reusable workflow implementations. Universal reusable
  behavior is called from `NWarila/.github`; stack-specific behavior is called
  from the runner template or framework by immutable SHA.
- `repo-hygiene.yaml` is the local caller that makes workflow pinning and
  privileged-trigger policy visible on this data-only runner without adding a
  local policy tree.
- Consumer workflow names are caller names (`pr-validation`, `security`,
  `repo-hygiene`, `release`, `drift-gate`, `terraform-deploy`, `auto-merge`),
  not framework reusable names.
