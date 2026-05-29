# Trivy Analysis

The Trivy Analysis workflow (`trivy.yaml`) scans your repository for different
types of vulnerabilities and misconfigurations using [Trivy]. See the [official
documentation][docs] for more information on using Trivy.

This workflow will run the Trivy scan, parse the results into GitHub
annotations, and optionally upload the SARIF file to GitHub[^ghas]. Any findings
from the Trivy scan will result in a failure of the workflow.

## Prerequisites

- A [`trivy.yaml` configuration file][config] at the root of your repository

## Usage

You can use the Trivy Analysis workflow in your own workflow by calling it as a
remote workflow:

```yaml
jobs:
  trivy:
    uses: codeforamerica/github-actions/.github/workflows/trivy.yaml@main
    with:
      path: ./
```

## Inputs

| Name   | Description                    | Required | Default |
| ------ | ------------------------------ | -------- | ------- |
| `path` | The path to the files to scan. | No       | `./`    |

## Outputs

| Name          | Description                                              |
| ------------- | -------------------------------------------------------- |
| `trivy.sarif` | The SARIF file containing the results of the Trivy scan. |

[config]: https://trivy.dev/docs/latest/references/configuration/config-file/
[docs]: https://trivy.dev/docs/latest/guide/
[trivy]: https://trivy.dev/

[^ghas]: SARIF uploads are only support on public repositories and private
  repositories with GitHub Advanced Security enabled.
