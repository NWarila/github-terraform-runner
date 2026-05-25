# Threat Model

## Scope

This runner manages repository inventory for the `NWarila` GitHub account. Its
main risks are unsafe inventory changes, private inventory exposure, privileged
workflow abuse, and deploy credentials being used outside the reviewed deploy
path.

## Guarantees

- **Inventory-only boundary.** The repository must not contain an executable
  Terraform module. Terraform logic lives in the pinned
  `github-terraform-framework` ref.
- **Private inventory boundary.** Real private repository YAML is fetched from
  S3 during deploy. Committed private data is limited to `.gitkeep` and
  public-safe test fixtures.
- **Pinned execution boundary.** Runner-template and framework reusable
  workflows are called by full commit SHA, and Renovate carries those pins
  forward.
- **Credential boundary.** Deploy uses GitHub OIDC for AWS and the configured
  fine-grained GitHub token secret. Static AWS access keys are not accepted.
- **Privileged workflow boundary.** `pull_request_target` is limited to
  trusted-bot auto-merge and uses the local `reusable-auto-merge.yaml` so the
  privileged surface remains inspectable.

## Out Of Scope

- Correctness of the GitHub provider implementation. That belongs to
  `github-terraform-framework`.
- Secrecy of data intentionally published under `terraform/public/`.
- Validation of real private inventory before it is uploaded to S3, beyond the
  deploy-time framework gate.
- Branch protection configuration outside the workflow files. Auto-merge
  relies on GitHub branch protection to define required checks.

## Failure Modes To Watch

- `template_ref`, runner-template `uses:`, and drift-gate `source-ref` moving
  to different commits.
- `framework_ref` and the framework deploy reusable SHA moving out of lockstep.
- Any new local `tools/`, `policies/`, or reusable workflow implementation
  beyond `reusable-auto-merge.yaml`.
- Any `pull_request_target` workflow that checks out PR-controlled content or
  reads PR-head refs.

Cross-reference: `SECURITY.md` (in `nwarila/.github` or
`nwarila-platform/.github`) defines the org-level reporting channel and
the org-wide scope boundary.
