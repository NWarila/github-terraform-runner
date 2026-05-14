# Architecture

## Module Boundary

This repository is a data-only runner for the NWarila GitHub account. It owns
repository inventory under `terraform/public/` and the empty
`terraform/private/` mount point for private inventory fetched during deploy.
The executable Terraform module lives in
`nwarila-platform/github-terraform-framework`.

## Validation Boundary

Pull request validation checks out the pinned framework ref, overlays
`terraform/public/` and the public-safe private fixture into the framework's
`terraform/repos/` tree, then runs the framework CI gate against that assembled
workspace.

## Deploy Boundary

Main-branch deploys call the framework reusable deploy workflow. That workflow
downloads private repository YAML from S3 into `terraform/private/`, overlays
both inventory folders into the framework, and runs Terraform with AWS OIDC and
the configured GitHub token.
