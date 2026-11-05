# Reusable GitHub Actions Workflows

This repository contains reusable GitHub Actions workflows shared across repositories. Centralizing common CI/CD tasks reduces duplication and keeps automation consistent.

## Available workflows

Workflows are stored in `.github/workflows/` and invoked using `workflow_call`.

### `rw-compute-version.yml`

Reads the latest SemVer version from a changelog (default: `CHANGELOG`), checks that the `Unreleased` section is empty, and computes version metadata from the GitHub ref and commit SHA.

**Input**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `changelog_path` | No | `CHANGELOG` | Path to the changelog file. |

**Outputs**

| Name | Description |
| --- | --- |
| `version` | Latest version from the changelog. |
| `ref_slug` | Lowercase, hyphenated branch or tag name. |
| `short_sha` | First seven characters of the commit SHA. |
| `image_tag` | `v<version>-<ref_slug>-<short_sha>` |
| `nuget_version` | Version without ref/SHA suffix. |
| `octopus_version` | `<version>-<ref_slug>-<short_sha>` |

Example:

```yaml
jobs:
  version:
    uses: pharmacist-75/infra-workflows/.github/workflows/rw-compute-version.yml@v1.0.0
    with:
      changelog_path: CHANGELOG

  build:
    needs: version
    runs-on: ubuntu-latest
    steps:
      - run: echo "Version: ${{ needs.version.outputs.version }}"
```

The changelog must contain a version line such as `1.0.0 - 2026-11-05`. The workflow fails if the file is missing or empty, no version is found, or the `Unreleased` section contains entries.

### `rw-create-github-release.yml`

Creates a GitHub release for the supplied version, using tag `v<version>` and automatically generated release notes.

**Input**

| Name | Required | Description |
| --- | --- | --- |
| `version` | Yes | Release version without the leading `v` (e.g. `1.0.0`). |

Example:

```yaml
jobs:
  release:
    permissions:
      contents: write
    uses: pharmacist-75/infra-workflows/.github/workflows/rw-create-github-release.yml@v1.0.0
    with:
      version: 1.0.0
```

The calling workflow must grant `contents: write` to allow release creation. The release workflow uses the GitHub CLI (`gh`) and the automatically provided `GITHUB_TOKEN`.

### `rw-create-tag.yml`

Creates and pushes an annotated Git tag named `v<version>` if it does not already exist. Existing tags are left unchanged.

**Input**

| Name | Required | Description |
| --- | --- | --- |
| `version` | Yes | Version without the leading `v` (e.g. `1.0.0`). |

**Output**

| Name | Description |
| --- | --- |
| `created` | `true` if the tag was created; `false` if it already existed. |

Example:

```yaml
jobs:
  tag:
    permissions:
      contents: write
    uses: pharmacist-75/infra-workflows/.github/workflows/rw-create-tag.yml@v1.0.0
    with:
      version: 1.0.0
```

### `rcw-release-tag.yml`

Orchestrates a release by computing the version, creating the Git tag, and creating a GitHub release **only when a new tag was created**. It calls the other three reusable workflows.

**Inputs:** None.

Example:

```yaml
jobs:
  release:
    permissions:
      contents: write
    uses: pharmacist-75/infra-workflows/.github/workflows/rcw-release-tag.yml@v1.0.0
```

**Note:** The orchestrator currently references the `niq-enterprise/dpidas-infra-workflows` repository at `@v4` internally. Update those references if you publish this workflow in a different repository or under a different release tag. The caller must grant `contents: write` for tag and release creation.

## Usage

Reference a published release tag (for example `@v1.0.0`) or pin a commit SHA for reproducibility.

## Development

Add new workflows under `.github/workflows/` and document their purpose, inputs, outputs, required permissions, and usage here. Changes should be reviewed through pull requests.
