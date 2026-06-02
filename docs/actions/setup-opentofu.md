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

### OpenTofu version

By default, this action installs a pinned version of OpenTofu and validates the
downloaded binary against known checksums. If you need a specific version, you
can override both the `tofu-version` and `checksums` inputs together.

> [!IMPORTANT]
> Always update `tofu-version` and `checksums` together. If only the version is
> changed, the checksum validation will fail. If only the checksums are changed,
> the wrong binary may be validated.

To find the checksums for a specific version, download the `SHA256SUMS` file
from the [OpenTofu releases page][releases] and include entries for the
architectures your runners use (typically `linux_amd64` for Ubuntu runners).

```yaml
- name: Setup OpenTofu
  uses: codeforamerica/github-actions/.github/actions/setup-opentofu@main
  with:
    config: service
    # Wrap the version in quotes to ensure it's treated as a string.
    tofu-version: "1.12.1"
    checksums: |
      1fc9af962e3632b7cd0ba27076cd9f1ced177567defe9e331ac37f5a40468575  tofu_1.12.1_linux_amd64.zip
```

## Inputs

| Name           | Description                                                                                                                   | Required | Default                       |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------- | -------- | ----------------------------- |
| `config`       | OpenTofu configuration to initialize.                                                                                         | Yes      |                               |
| `cache-key`    | Optional unique identifier to add to the cache key.                                                                           | No       | `""`                          |
| `checksums`    | Newline-separated list of checksums to validate the downloaded OpenTofu binary. Must be updated together with `tofu-version`. | No       | _(pinned to default version)_ |
| `configs-path` | Path to the OpenTofu configurations for your repository.                                                                      | No       | `./tofu/configs`              |
| `tofu-version` | OpenTofu version to install. Must be updated together with `checksums`.                                                       | No       | `"1.12.1"`                    |

## Outputs

_This action has no outputs._

[releases]: https://github.com/opentofu/opentofu/releases
