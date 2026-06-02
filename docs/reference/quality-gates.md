# Quality Gates

This runner is a consumer, so most gates are delegated to the pinned
`NWarila/.github`, `terraform-runner-template`, and `github-terraform-framework` reusable
workflows. The local repository owns inventory, workflow pins, deploy inputs,
public-safe fixtures, and runner-specific documentation.

> **Enforcement note.** This repo has no `required_status_checks` ruleset and no
> classic branch protection, so the CI checks below are **not mechanical merge
> gates**: a red check surfaces on the PR but does not by itself block merge. The
> actual merge gate is the Pull Request Gate ruleset (1 approving review +
> code-owner review + thread resolution, squash-only). "Expected" means a check is
> expected to be green before a reviewer merges, not that GitHub will refuse the
> merge while it is red.

| Role | Meaning | When it runs |
| --- | --- | --- |
| Expected | Runs on PRs and is expected to be green before merge. Reported on the PR; not a mechanical merge gate (see enforcement note). | `pull_request` (+ `merge_group`/`push` where the workflow declares them) |
| Scheduled | Periodic posture telemetry. Does not block PRs. | `schedule` triggers |
| Release | Runs at release-cut time; gates release-asset publishing inside `release.yaml`. | `release.yaml` |
| Advisory | Surfaces signal without making the scan result the gate. | Explicit `continue-on-error` publishing steps in delegated reusables |

## Gate Inventory

| Gate | Source | Role | Notes |
| --- | --- | --- | --- |
| Runner validation | `pr-validation.yaml` -> `terraform-runner-template/reusable-terraform-validation.yaml` | Expected | Runs runner contract checks, workflow/caller validation, lint, and framework validation against this repo's inventory overlay. |
| Framework validation | `pr-validation.yaml` via pinned `github-terraform-framework` `framework_ref` | Expected | Assembles framework plus runner inventory and public-safe private fixtures before running framework CI. |
| Drift gate (org-baseline + runner-template) | `drift-gate.yaml` -> `NWarila/drift-gate` (two jobs) | Expected | Byte-compares this repo against (a) the org baseline files from `NWarila/.github` and (b) the contract-critical scaffold from the pinned `terraform-runner-template`. Runs on `pull_request` + `workflow_dispatch` only (no `merge_group`/`push`). |
| Security scan | `security.yaml` -> `NWarila/.github/reusable-iac-security.yaml` | Expected | Runs IaC, secret, and Actions posture scans through the org-owned implementation. |
| CodeQL | `security.yaml` -> `NWarila/.github/reusable-codeql.yaml` | Expected | Static analysis is delegated to the org-owned reusable. SARIF upload may be advisory depending on GitHub Security availability. |
| OpenSSF Scorecard | `security.yaml` -> `NWarila/.github/reusable-scorecard.yaml` | Scheduled / branch-protection / manual | Skipped on PR and merge queue because private-repo GraphQL access is not reliable for PR gating. |
| Repo hygiene | `repo-hygiene.yaml` -> `NWarila/.github/reusable-repo-hygiene.yaml` | Expected / scheduled | Enforces SHA-pinned workflow refs and privileged-trigger safety without local policy tooling. |
| Terraform deploy | `terraform-deploy.yaml` -> `github-terraform-framework/reusable-terraform-deploy.yaml` | Push to main / manual | Applies reviewed inventory using AWS OIDC, S3 backend inputs, and the fine-grained GitHub token secret. |
| Release Please | `release.yaml` -> `NWarila/.github/reusable-release-please.yaml` | Release | Creates release PRs/releases when explicitly enabled or manually dispatched. |
| Release evidence | `release.yaml` -> `terraform-framework-template/reusable-release-evidence.yaml` | Release | Publishes runner release evidence, SBOM, and attestations. |
| Auto-merge | `auto-merge.yaml` -> `NWarila/.github/reusable-auto-merge.yaml` | Not a gate | Enables auto-merge for trusted bot PRs once the Pull Request Gate review requirements are met. No required status checks are configured, so it does not wait for CI. Privileged-trigger safety is flagged by the advisory repo-hygiene check. |

## Local Ownership

This repository intentionally does not have a local `tools/verify.py`,
`Makefile`, OPA policy tree, or contract fixture harness. Those are template
maintainer surfaces. Adding them here would bypass the runner contract and
create a second source of truth.

## Adding A New Gate

1. Prefer adding reusable implementation in `NWarila/.github`,
   `terraform-runner-template`, or `github-terraform-framework`, depending on
   whether the gate is universal, runner-specific, or framework-specific.
2. Add only the caller or repo-specific input here.
3. Pin every remote reusable and body ref to a 40-character SHA.
4. Document the gate in this file and in the owning template's quality-gates
   reference.
