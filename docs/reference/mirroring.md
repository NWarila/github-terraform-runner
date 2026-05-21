# Mirroring And Runner Baseline

This runner mirrors only the stable, byte-identical scaffold declared by the
pinned `terraform-runner-template` baseline. Template-maintainer files such as
contract harnesses, local OPA policy tests, integration fixtures, and
`tools/verify.py` do not belong in this repository.

## What Is Mirrored

The drift gate enforces only byte-identical scaffold that should never vary in
runner repos: formatting config, layout sentinels, the PR template, license,
and the local privileged auto-merge reusable. The source of truth is the pinned
`source-ref` in `.github/workflows/drift-gate.yaml`.

Workflow caller shape is enforced by the runner-template contract during
`pr-validation.yaml`. Security and release callers must delegate to
`terraform-runner-template` reusable workflows by immutable SHA; the only local
reusable workflow implementation allowed here is `reusable-auto-merge.yaml`.

## What This Repo Owns

This repo owns:

- `terraform/public/*.yml`
- `terraform/private/.gitkeep`
- `tests/fixtures/terraform/private/*.yml`
- workflow pins and deploy inputs
- runner-specific documentation

## Update Rule

When the runner template changes, merge the template change first, then bump
both runner template pins in this repo to the same template commit SHA:

- `pr-validation.yaml` `uses:` and `template_ref`
- `drift-gate.yaml` `source-ref`
