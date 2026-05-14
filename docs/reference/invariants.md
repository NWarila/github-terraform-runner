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
