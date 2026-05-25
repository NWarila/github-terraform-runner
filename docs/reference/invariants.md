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
- The only local reusable workflow implementation allowed here is
  `reusable-auto-merge.yaml`, because privileged `pull_request_target` behavior
  must be statically inspectable in the consumer repo.
- Consumer workflow names are caller names (`pr-validation`, `security`,
  `release`, `drift-gate`, `terraform-deploy`, `auto-merge`), not framework
  reusable names.
