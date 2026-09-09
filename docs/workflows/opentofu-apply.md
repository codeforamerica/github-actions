# OpenTofu Apply

The OpenTofu Apply workflow (`opentofu-apply.yaml`) runs `tofu apply` against
the specified configuration layer and environment. It first runs the [plan
workflow][plan-workflow] and uses the resulting plan file to apply the changes.

## Prerequisites

- An IAM role in the target AWS account (even if AWS is only used for the state
  backend) with permissions necessary to manage your resources
- An OIDC connection between your GitHub repository and the target AWS account
- A Doppler project with the necessary secrets and variables for your OpenTofu
  configuration, and synced to your GitHub environments
- A Doppler Service Account with access to the project and environments
- A Doppler [Service Account Identity][doppler-identity] for your GitHub
  repository

## Usage

You can use the OpenTofu Apply workflow in your own workflow by calling it as
a remote workflow. You will need to pass the `AWS_ROLE_ARN` and
`DOPPLER_OIDC_IDENTITY` secrets to the workflow, along with required inputs.

Optionally, you can pass a `distinct_id` which will be printed to the logs as
the first step of the workflow. This can be used by remote systems, such as
workflows in a different repository, to find and wait for the workflow to
complete. If no `distinct_id` is provided, the GitHub run ID will be used.

```yaml
jobs:
  apply:
    uses: codeforamerica/github-actions/.github/workflows/opentofu-apply.yaml@main
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      DOPPLER_OIDC_IDENTITY: ${{ secrets.DOPPLER_OIDC_IDENTITY }}
    with:
      config: ${{ inputs.config }}
      distinct_id: ${{ inputs.distinct_id || github.run_id }} # Optional
      environment: ${{ inputs.environment }}
```

## Inputs

| Name           | Description                                              | Required | Default                |
| -------------- | -------------------------------------------------------- | -------- | ---------------------- |
| `application`  | Application matching a spec in the selected configuration. | No       | `""`                  |
| `config`       | The OpenTofu configuration layer to apply.               | Yes      | n/a                    |
| `configs-path` | Path to the OpenTofu configurations for your repository. | No       | `./tofu/configs`       |
| `distinct_id`  | Optional unique identifier to print to the logs.         | No       | `${{ github.run_id }}` |
| `environment`  | GitHub environment to run the apply on.                  | No       | `development`          |

## Secrets

| Name                    | Description                                                  | Required |
| ----------------------- | ------------------------------------------------------------ | -------- |
| `AWS_ROLE_ARN`          | ARN of the IAM role to assume before running the apply.      | Yes      |
| `DOPPLER_OIDC_IDENTITY` | Doppler Service Account Identity for your GitHub repository. | Yes      |

## Outputs

_This action has no outputs._

[doppler-identity]: ../actions/doppler-oidc-token.md#configuration
[plan-workflow]: ./opentofu-plan.md
