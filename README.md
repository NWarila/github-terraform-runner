# NWarila/github-terraform-runner

GitHub-as-code deployer for the [NWarila](https://github.com/NWarila) GitHub
user account. Owns repository inventory under `terraform/` and delegates the
actual `terraform apply` to the
[github-terraform-framework](https://github.com/nwarila-platform/github-terraform-framework)
reusable workflow.

This repository is a runner under the
[NWarila/terraform-runner-template](https://github.com/NWarila/terraform-runner-template)
contract. It contains no Terraform module code of its own; validation,
contract checks, security scanning, and release evidence are delegated to
`terraform-runner-template`. Trusted-bot auto-merge is the one local reusable
workflow exception because it runs under `pull_request_target` and must remain
fully inspectable by the privileged-workflow static analyzer.

## Layout

```text
terraform/
  public/    YAML definitions for public repos under NWarila
  private/   Empty in-repo (gitkeep only); fetched from S3 at deploy time
             (Personal.yml, Resume.yml, github-sandbox.yml)
tests/
  fixtures/terraform/private/
             Public-safe private fixtures used by pr-validation
.github/workflows/
  pr-validation.yaml     checks out the pinned framework, overlays this runner's
                         inventory, and runs framework CI
  security.yaml          calls template-owned security reusable workflows by SHA
  release.yaml           calls template-owned release reusable workflows by SHA
  auto-merge.yaml        local privileged auto-merge caller
  reusable-auto-merge.yaml
                         local privileged implementation inspected by the
                         static analyzer
  terraform-deploy.yaml  plans and applies on main using the framework deploy
                         reusable, AWS OIDC, and repo secrets
```

## How A Change Lands

1. Edit YAML under `terraform/public/`, or upload reviewed private YAML to S3.
2. PR validation assembles framework plus this runner data plus the public-safe
   private fixture, then runs contract, lint, security, and Terraform plan
   gates.
3. After merge, `terraform-deploy.yaml` applies on `main`.

Renovate keeps `framework_ref`, the framework reusable SHA, and the
runner-template SHA current. Trusted-bot PRs auto-merge once required checks
pass; human PRs follow normal review.

The complete gate inventory lives in
[`docs/reference/quality-gates.md`](docs/reference/quality-gates.md).
