# Reusable GitHub Actions Workflows

This repository provides a central place for reusable GitHub Actions workflows.

The goal is to share common CI/CD automation across repositories, reduce duplication, and keep workflow behavior consistent and maintainable.

## Repository structure

Reusable workflows will be stored in `.github/workflows/` and can be called from other repositories using GitHub Actions' `workflow_call` trigger.

## Usage

Once workflows are available, consuming repositories can reference them using the following syntax:

```yaml
jobs:
  example:
    uses: <owner>/<repository>/.github/workflows/<workflow>.yml@<ref>
```

Replace the placeholders with the repository owner (`pharmacist-75`), repository name (`infra-workflows`), workflow filename, and a version tag or commit SHA.

## Development

Workflows will be added incrementally. Each workflow will include documentation describing its purpose, required inputs, secrets, permissions, and usage.

## Status

This is the initial repository setup. No reusable workflows are included yet.
