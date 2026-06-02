# OpenTofu Linting (TFLint)

The OpenTofu Linting workflow (`tflint.yaml`) runs [TFLint] against your
repository. This will help you catch potential errors, avoid common mistakes,
and enforce best practices.

> [!IMPORTANT]
> TFLint does not officially support OpenTofu. However, with a few adjustments,
> it can be used to lint your OpenTofu code. Make sure to reference the [sample
> configuration][sample] below for the best results.

This workflow will run the TFLint scan recursively at the specified path, parse
the results into GitHub annotations, and optionally upload the SARIF file to
GitHub[^ghas]. Any findings from the TFLint scan will result in a failure of the
workflow.

## Prerequisites

- A [`.tflint.hcl` configuration file][config] at the root of your repository

### Sample configuration file

The following is a sample TFLint configuration file that should work for most
repositories. You can find the latest available versions of the AWS plugin at
the [terraform-linters/tflint-ruleset-aws][aws-plugin] repository.

```hcl title=".tflint.hcl"
# Only enable the AWS ruleset if you're using AWS.
plugin "aws" {
  enabled = true
  version = "0.47.0"
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

plugin "terraform" {
  preset = "all"
  enabled = true
}

# TFLint doesn't understand the provider for_each syntax introduced with
# OpenTofu 1.9, so we need to disable these rules so it doesn't error out.
rule "terraform_required_providers" {
  enabled = false
}
rule "terraform_unused_required_providers" {
  enabled = false
}
```

## Usage

You can use the OpenTofu Linting workflow in your own workflow by calling it as
a remote workflow:

```yaml
jobs:
  tflint:
    uses: codeforamerica/github-actions/.github/workflows/tflint.yaml@main
    with:
      path: ./
```

## Inputs

| Name   | Description                                 | Required | Default |
| ------ | ------------------------------------------- | -------- | ------- |
| `path` | The path to the files to lint, recursively. | No       | `./`    |

## Outputs

| Name           | Description                                               |
| -------------- | --------------------------------------------------------- |
| `tflint.sarif` | The SARIF file containing the results of the TFLint scan. |

[aws-plugin]: https://github.com/terraform-linters/tflint-ruleset-aws
[config]: https://github.com/terraform-linters/tflint/blob/master/docs/user-guide/config.md
[sample]: #sample-configuration-file
[tflint]: https://github.com/terraform-linters/tflint

[^ghas]: SARIF uploads are only support on public repositories and private
  repositories with GitHub Advanced Security enabled.
