# Release Gates

PRs to `main` run the following checks. They are **expected to be green before
merge but are advisory, not mechanical merge gates**: this repo has no
`required_status_checks` ruleset, so a red check does not by itself block merge
(the Pull Request Gate review rules do). See
[`quality-gates.md`](quality-gates.md) for the enforcement model.

- `pr-validation.yaml`: calls the pinned
  `NWarila/terraform-runner-template` reusable, overlays this runner's
  `terraform/public/` inventory and public-safe private fixture into the pinned
  framework, then runs the framework quality gate.
- `drift-gate.yaml`: verifies only byte-identical scaffold files from the
  pinned runner template.
- `security.yaml`: calls org-owned reusable workflows for IaC scanning, CodeQL,
  and Scorecard.
- `repo-hygiene.yaml`: calls the org-owned reusable policy check for workflow
  pinning and privileged-trigger safety.

This repo does not have a local `make ci` gate. The executable Terraform module
and its tests live in `nwarila-platform/github-terraform-framework`.
