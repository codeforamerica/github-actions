# Shared GitHub Action Workflows

This repository contains a collection of shared GitHub Action workflows that can
be used in your own repositories.

By creating and maintaining these workflows in a single repository, we can
reduce duplication and ensure consistency across our repositories. If you have a
workflow that you think would be useful to share, please open a pull request to
make a contribution.

## Usage

To use a workflow in your own repository, you can add the following to your
`.github/workflows/` directory:

```yaml
jobs:
  workflow-name:
    uses: codeforamerica/github-actions/.github/workflows/workflow-name.yaml@main
    with:
      input-name: input-value
```

The filename and inputs you require will depend on the workflow(s) you're using.

## Workflows

| Workflow              | Description                                                                                                |
| --------------------- | ---------------------------------------------------------------------------------------------------------- |
| `security-scan.yml`   | Runs a collection of security scans against your repository.                                               |
| [`tflint.yaml`]       | Runs a TFLint scan against your OpenTofu (or Terraform) code, and optionally upload the results to GitHub. |
| [`trivy.yaml`][trivy] | Runs a Trivy security scan against your repository, and optionally upload the results to GitHub.           |

[tflint]: docs/workflows/tflint.md
[trivy]: docs/workflows/trivy.md
