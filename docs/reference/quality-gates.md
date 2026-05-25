# Quality Gates

This runner is a consumer, so most gates are delegated to the pinned
`terraform-runner-template` and `github-terraform-framework` reusable
workflows. The local repository owns inventory, workflow pins, deploy inputs,
and the privileged auto-merge reusable.

| Role | Meaning | When it runs |
| --- | --- | --- |
| Blocking | Required for PR merge to `main`. Failure blocks the PR. | `pull_request`, `merge_group`, and protected push paths |
| Scheduled | Periodic posture telemetry. Does not block PRs. | `schedule` triggers |
| Release | Runs at release-cut time. Failure blocks release assets. | `release.yaml` |
| Advisory | Surfaces signal without making the scan result the gate. | Explicit `continue-on-error` publishing steps in delegated reusables |

## Gate Inventory

| Gate | Source | Role | Notes |
| --- | --- | --- | --- |
| Runner validation | `pr-validation.yaml` -> `terraform-runner-template/reusable-terraform-validation.yaml` | Blocking | Runs runner contract checks, workflow/caller validation, lint, and framework validation against this repo's inventory overlay. |
| Framework validation | `pr-validation.yaml` via pinned `github-terraform-framework` `framework_ref` | Blocking | Assembles framework plus runner inventory and public-safe private fixtures before running framework CI. |
| Template drift gate | `drift-gate.yaml` -> `NWarila/drift-gate` | Blocking | Byte-compares stable runner scaffold against the pinned `terraform-runner-template` baseline. |
| Security scan | `security.yaml` -> `terraform-runner-template/reusable-iac-security.yaml` | Blocking | Runs IaC, secret, and Actions posture scans through the template-owned implementation. |
| CodeQL | `security.yaml` -> `terraform-runner-template/reusable-codeql.yaml` | Blocking | Static analysis is delegated to the template-owned reusable. SARIF upload may be advisory depending on GitHub Security availability. |
| OpenSSF Scorecard | `security.yaml` -> `terraform-runner-template/reusable-scorecard.yaml` | Scheduled / branch-protection / manual | Skipped on PR and merge queue because private-repo GraphQL access is not reliable for PR gating. |
| Terraform deploy | `terraform-deploy.yaml` -> `github-terraform-framework/reusable-terraform-deploy.yaml` | Push to main / manual | Applies reviewed inventory using AWS OIDC, S3 backend inputs, and the fine-grained GitHub token secret. |
| Release Please | `release.yaml` -> `terraform-runner-template/reusable-release-please.yaml` | Release | Creates release PRs/releases when explicitly enabled or manually dispatched. |
| Release evidence | `release.yaml` -> `terraform-runner-template/reusable-release-evidence.yaml` | Release | Publishes runner release evidence, SBOM, and attestations. |
| Auto-merge | `auto-merge.yaml` -> local `reusable-auto-merge.yaml` | Not a gate | Enables auto-merge only for trusted bot PRs after branch protection checks pass. Kept local for privileged-workflow inspection. |

## Local Ownership

This repository intentionally does not have a local `tools/verify.py`,
`Makefile`, OPA policy tree, or contract fixture harness. Those are template
maintainer surfaces. Adding them here would bypass the runner contract and
create a second source of truth.

## Adding A New Gate

1. Prefer adding reusable implementation in `terraform-runner-template` or
   `github-terraform-framework`.
2. Add only the caller or repo-specific input here.
3. Pin every remote reusable and body ref to a 40-character SHA.
4. Document the gate in this file and in the owning template's quality-gates
   reference.
