# Release Gates

PRs to `main` must pass:

- `pr-validation.yaml`: calls the pinned
  `NWarila/terraform-runner-template` reusable, overlays this runner's
  `terraform/public/` inventory and public-safe private fixture into the pinned
  framework, then runs the framework quality gate.
- `drift-gate.yaml`: verifies only byte-identical scaffold files from the
  pinned runner template.
- `security.yaml`: runs the local reusable security callers for IaC scanning,
  CodeQL, and Scorecard.

This repo does not have a local `make ci` gate. The executable Terraform module
and its tests live in `nwarila-platform/github-terraform-framework`.
