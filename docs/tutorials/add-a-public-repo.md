# Tutorial: Add a New Public Repository to the Inventory

This tutorial walks through adding a new public GitHub repository under the
`NWarila` account to the `github-terraform-runner` inventory. After completing
it the new repository will be created and governed by Terraform.

## Prerequisites

- Write access to this repository.
- Familiarity with the YAML schema used by `nwarila-platform/github-terraform-framework`.
- A local clone of this repository on a branch off `main`.

## Steps

### 1. Create the inventory file

Add a `.yml` file under `terraform/public/`. Use the repository name as the
filename.

```text
terraform/public/my-new-repo.yml
```

Populate the file with the repository definition. The minimum required fields
are `name` and `visibility`. For example:

```yaml
name: my-new-repo
visibility: public
description: Short description of the repository.
codeowners: |
  * @NWarila
```

Consult the framework schema for the full list of accepted fields.

### 2. Verify the YAML is tracked by git

The `.gitignore` in this repository uses a default-deny strategy. Every new
file must match an existing allowlist glob. Public inventory files are covered
by:

```
!/terraform/public/*.yml
```

Run `git status` after creating the file to confirm it appears as an untracked
file rather than being silently ignored.

```sh
git status --short
```

The new file should appear with a `?` prefix. If it is absent, check whether
the filename uses `.yaml` instead of `.yml` and rename accordingly.

### 3. Open a pull request

Push the branch and open a PR against `main`. The `pr-validation.yaml`
workflow will:

1. Check out the pinned `github-terraform-framework` at the pinned `framework_ref`.
2. Overlay your local inventory on top of the framework workspace.
3. Run `terraform validate` and `terraform plan` against the merged workspace.

Review the plan output in the PR checks to confirm the new repository appears
as an `+ create` resource with the expected attributes.

### 4. Merge

Once the PR is approved and the plan looks correct, merge via squash. The
`terraform-deploy.yaml` workflow triggers on push to `main` and applies the
plan, creating the repository in GitHub.

## What Happens Next

Terraform creates the repository with the settings you declared. Subsequent
changes to the inventory file follow the same PR-then-apply cycle. Deleting
the file from inventory will cause `terraform plan` to show the repository as
a `- destroy` resource; that change requires an explicit review before merge.

## Related Reference

- Workflow layout: [README.md](../../README.md)
- Deploy sequence: [docs/diagrams/deploy-overlay-sequence.mmd](../diagrams/deploy-overlay-sequence.mmd)
- Quality gates: [docs/reference/quality-gates.md](../reference/quality-gates.md)
