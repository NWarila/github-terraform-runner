# Testing Strategy

This runner is intentionally thin: it owns GitHub repository inventory and
delegates executable Terraform behavior to the pinned
`nwarila-platform/github-terraform-framework` ref. Testing therefore focuses
on proving that the inventory can be safely assembled with that framework and
that the runner still satisfies the `terraform-runner-template` contract.

## What The Tests Cover

- **Runner contract.** `pr-validation.yaml` calls the pinned
  `terraform-runner-template` reusable with `run_contract_check: true`. The
  contract verifies required files, forbidden local template-maintainer
  surfaces, SHA-pinned workflow references, safe deploy/auth patterns, and
  template drift.
- **Framework integration.** Pull request validation checks out the pinned
  `github-terraform-framework`, overlays `terraform/public/` plus the
  public-safe private fixture into the framework runtime tree, and runs the
  framework quality gate against that assembled workspace.
- **Inventory shape.** YAML linting runs through pre-commit and the delegated
  reusable validation path. The framework gate is responsible for schema and
  Terraform-level validation after overlay.
- **Privileged workflow safety.** The only local reusable workflow is
  `reusable-auto-merge.yaml`; the runner-template contract and security gate
  keep `pull_request_target` isolated from PR-controlled checkout paths.
- **Security posture.** `security.yaml` delegates IaC, secret, Actions,
  CodeQL, and Scorecard checks to template-owned reusable workflows pinned by
  SHA.

## What The Tests Do Not Cover

- **Private inventory contents.** Real private repository YAML is fetched from
  S3 during deploy and is not committed. PR validation uses the public-safe
  fixture under `tests/fixtures/terraform/private/`.
- **A live Terraform apply during PRs.** Pull requests validate assembly and
  plan-time behavior. Main-branch deploy is the apply path.
- **Framework implementation details.** Provider resources, module tests, OPA
  resource policy, and Terraform unit/integration tests live in
  `github-terraform-framework`.
- **GitHub API side effects in PR validation.** The PR path does not mutate
  repositories. Mutation happens only through `terraform-deploy.yaml` after
  merge, using reviewed workflow inputs and secrets.
