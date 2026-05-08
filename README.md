# NWarila/github-terraform-runner

GitHub-as-code deployer for the [NWarila](https://github.com/NWarila) GitHub
user account. Owns the inventory of repositories under `repos/` and
delegates the actual `terraform apply` to the
[github-terraform-framework](https://github.com/nwarila-platform/github-terraform-framework)
reusable workflow.

This repository is a *runner* under the
[NWarila/terraform-template](https://github.com/NWarila/terraform-template)
contract. It contains no Terraform module code of its own; every gate
(validation, security scan, CodeQL, scorecard, sync, release, auto-merge)
runs through reusable workflows from terraform-template, and the deploy
runs through `nwarila-platform/github-terraform-framework`'s
`reusable-terraform-deploy` workflow.

## Layout

```
repos/
  public/    YAML definitions for public repos under NWarila
  private/   Empty in-repo (gitkeep only); fetched from S3 at deploy time
             (Personal.yml, Resume.yml, github-sandbox.yml)
tests/
  fixtures/  Public-safe fixtures used by pr-validation
.github/workflows/
  pr-validation.yaml     end-to-end CI: checks out the framework at the
                         pinned SHA, overlays this runner's repos/, runs
                         `make ci` against the assembled tree
  terraform-deploy.yaml  the apply path: plans and applies on push to main,
                         with three named private YAMLs s3-cp'd at runtime
  ...                    universal callers (security, codeql, scorecard,
                         release-please, auto-merge, template-sync)
```

## How a change lands

1. Edit a YAML under `repos/public/` (or upload a new private YAML to S3).
2. PR Validation runs end-to-end: framework + this PR's data + the
   public-safe `tests/fixtures/` private overlay → must pass contract,
   lint, security, and `terraform plan`.
3. After merge, `terraform-deploy.yaml` applies on `main`.

Renovate keeps `framework_ref` and the deploy-reusable SHA in lockstep with
the framework's `main`. Trusted-bot PRs auto-merge once required checks
pass; human PRs follow normal review.
