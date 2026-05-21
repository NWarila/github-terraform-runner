# Develop This Runner

## Local Setup

This repo is a thin deployer. It owns repository inventory YAML under
`terraform/public/` and delegates Terraform validation and apply behavior to
pinned reusable workflows.

For local editing, install:

- Python 3.12 with `pre-commit`
- actionlint, if you want local workflow linting outside pre-commit

Terraform, TFLint, OPA, and terraform-docs run in the consumed framework during
PR validation; this repo does not keep a local framework-shaped `tools/`
harness.

## Development Loop

1. Edit `terraform/public/*.yml`, or update the public-safe private fixture in
   `tests/fixtures/terraform/private/` when PR validation needs representative
   private input.
2. Run `pre-commit run --all-files` for local YAML, workflow, and Markdown
   checks.
3. Open a PR and let `pr-validation.yaml` assemble the pinned framework with
   this runner's inventory.

## Before Opening A PR

Confirm these pins move together when bumping templates:

- `.github/workflows/pr-validation.yaml` `uses:` and `template_ref`
- `.github/workflows/drift-gate.yaml` `source-ref`
- `.github/workflows/pr-validation.yaml` `framework_ref`
- `.github/workflows/terraform-deploy.yaml` reusable SHA and `framework_ref`
