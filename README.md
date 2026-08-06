# Shared GitHub Action Workflows

This repository contains a collection of shared GitHub Action workflows and
composite actions that can be used in your own repositories.

By creating and maintaining these resources in a single repository, we can
reduce duplication and ensure consistency across our repositories. If you have a
workflow or action that you think would be useful to share, please open a pull
equest to make a contribution.

## Workflows

Workflows are full-featured CI/CD workflows that are used to automate your
processes. They can be used to run tests, lint your code, deploy your
application, and much more.

To use a workflow in your own repository, you can add a workflow to your
`.github/workflows/` directory, similar to the following:

```yaml title=".github/workflows/workflow-name.yaml"
jobs:
  workflow-name:
    uses: codeforamerica/github-actions/.github/workflows/workflow-name.yaml@main
    with:
      input-name: input-value
```

The filename and inputs you require will depend on the workflow(s) you're using.

### Available workflows

| Workflow                                         | Description                                                                                                                      |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| `security-scan.yml`                              | Runs a collection of security scans against your repository.                                                                     |
| [`opentofu-apply.yaml`][workflow-opentofu-apply] | Runs `tofu apply` against the specified configuration layer and environment.                                                     |
| [`opentofu-plan.yaml`][workflow-opentofu-plan]   | Runs `tofu plan` against the specified configuration layer and environment, and captures the resulting plan file as an artifact. |
| [`tflint.yaml`][workflow-tflint]                 | Runs a TFLint scan against your OpenTofu (or Terraform) code, and optionally upload the results to GitHub.                       |
| [`trivy.yaml`][workflow-trivy]                   | Runs a Trivy security scan against your repository, and optionally upload the results to GitHub.                                 |

## Composite actions

Composite actions are actions that are composed of multiple steps. They aren't
entire workflows, but encapsulate common tasks that are used across workflows.

To use a composite action in your own repository, you can add the following to
your own workflow file, similar to the following:

```yaml title=".github/workflows/workflow-name.yaml"
jobs:
  workflow-name:
    runs-on: ubuntu-latest
    steps:
      # Add steps to your workflow...
      - name: Use a composite action
        uses: codeforamerica/github-actions/.github/actions/action-name@main
        with:
          input-name: input-value
      # ...
```

The action name and inputs you require will depend on the action(s) you're
using.

### Available actions

| Action                                                        | Description                                                                         |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| [`doppler-oidc-token`][action-doppler-oidc-token]             | Retrieves a single-use OIDC token for Doppler.                                      |
| [`opentofu-changed-modules`][action-opentofu-changed-modules] | Finds all OpenTofu modules that have changed in a pull request or push to a branch. |
| [`security-features`][action-security-features]               | Checks if the repository has access to different GitHub security features.          |
| [`setup-opentofu`][action-setup-opentofu]                     | Sets up OpenTofu and related environment variables.                                 |

[action-security-features]: docs/actions/security-features.md
[action-setup-opentofu]: docs/actions/setup-opentofu.md
[action-opentofu-changed-modules]: docs/actions/opentofu-changed-modules.md
[action-doppler-oidc-token]: docs/actions/doppler-oidc-token.md
[composite-actions]: https://docs.github.com/en/actions/tutorials/create-actions/create-a-composite-action
[workflow-opentofu-apply]: docs/workflows/opentofu-apply.md
[workflow-opentofu-plan]: docs/workflows/opentofu-plan.md
[workflow-tflint]: docs/workflows/tflint.md
[workflow-trivy]: docs/workflows/trivy.md
