# Security Features

The Security Features action (`security-features/action.yaml`) checks if the
repository has access to different GitHub security features. Some features are
only available on public repositories and/or require an additional paid plan.

Use this workflow when you have another workflow that requires access to these
security features. You can use the outputs of this action to skip certain steps,
or entire jobs, when required features are not available.

## Usage

```yaml
jobs:
  my-job:
    runs-on: ubuntu-latest
    steps:
      - name: Check available security features
        id: security-features
        uses: codeforamerica/github-actions/.github/actions/security-features@main
```

You can then use the outputs of this action in your workflow. For example, to
only upload a SARIF file (name `results.sarif` in this example) to GitHub when
SARIF uploads are supported, you can do the following:

```yaml
      - name: Upload SARIF file if supported
        if: always() && steps.security-features.outputs.sarif == 'true'
        uses: github/codeql-action/upload-sarif@v4
        with:
          sarif_file: results.sarif
```

## Inputs

_This action has no inputs._

## Outputs

| Name     | Description                                                        |
| -------- | ------------------------------------------------------------------ |
| `codeql` | Whether or not [CodeQL] analysis is supported.                     |
| `ghas`   | Whether or not [GitHub Advanced Security][ghas] (GHAS) is enabled. |
| `public` | Whether or not the repository is public.                           |
| `sarif`  | Whether or not SARIF uploads are supported.                        |

[codeql]: https://docs.github.com/en/code-security/concepts/code-scanning/codeql/about-code-scanning-with-codeql
[ghas]: https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security
[sarif]: https://docs.github.com/en/code-security/reference/code-scanning/sarif-files/sarif-support-for-code-scanning
