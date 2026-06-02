# Setup OpenTofu

The Setup OpenTofu action (`setup-opentofu/action.yaml`) sets up OpenTofu for
use in your workflow.

This action will install OpenTofu, set up environment variables, and initialize
the OpenTofu configuration. Downloaded providers and modules will be cached to
reduce future run times.

## Usage

```yaml
jobs:
  my-job:
    runs-on: ubuntu-latest
    steps:
      - name: Setup OpenTofu
        uses: codeforamerica/github-actions/.github/actions/setup-opentofu@main
        env:
          # Set any variables and secrets that your configuration uses.
          DD_API_KEY: ${{ secrets.DD_API_KEY }}

          # Make sure OpenTofu inputs use full uppercase names and the
          # `TF_VAR_` prefix.
          TF_VAR_PROJECT: ${{ vars.TF_VAR_PROJECT }}
          TF_VAR_VPC_CIDR: ${{ secrets.TF_VAR_VPC_CIDR }}
        with:
          config: service
```

### Configuration

OpenTofu reads inputs from environment variables in the format `TF_VAR_<name>`,
where `<name>` matches the name of the input. This variable is case-sensitive.
Since we use lowercase names for our OpenTofu inputs, this results in mixed-case
environment variables. However, Doppler and GitHub Actions don't support
mixed-case variables and secrets.

Additionally, OpenTofu doesn't support optional inputs through environment
variables. For example, `TF_VAR_project=""` will be set the `project` input to
an empty string. The only way to natively support optional inputs is to leave
the environment variable unset.

To work around both of these limitations, this action converts any
`TF_VAR_<name>` environment variables to the proper case, and ignores any that
have an empty value.

> [!NOTE]
> This does not leak any values that may be set as secrets. GitHub Actions will
> automatically mask any secrets in the logs.

In your calling workflow, simply set the environment variables for your OpenTofu
inputs in the formate `TF_VAR_<NAME>` (all uppercase). This action will handle
the conversion and optional inputs.

## Inputs

| Name           | Description                                              | Required | Default          |
| -------------- | -------------------------------------------------------- | -------- | ---------------- |
| `config`       | OpenTofu configuration to initialize.                    | Yes      |                  |
| `cache-key`    | Optional unique identifier to add to the cache key.      | No       | `""`             |
| `configs-path` | Path to the OpenTofu configurations for your repository. | No       | `./tofu/configs` |

## Outputs

_This action has no outputs._
