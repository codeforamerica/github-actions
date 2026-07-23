# Apply OpenTofu formatting

The OpenTofu formatting workflow (`tofu-fmt.yaml`) runs [`tofu fmt`][fmt]
against your repository to keep your OpenTofu (or Terraform) code in a
consistent, canonical style.

Rather than failing when it finds unformatted code, this workflow **applies**
the formatting and commits the fixes back to the branch, so you never have to
run the formatter by hand. It follows the same pattern as our Ruby lint
autofix workflows.

The commit is skipped on `main`: that branch is a merge target and is typically
protected, so formatting is corrected on feature branches before merge. The fix
is committed with the default `GITHUB_TOKEN`, which does not re-trigger
workflows, so the workflow cannot loop.

> [!NOTE]
> `tofu fmt` only formats `*.tf` and `*.tfvars` files. It does not touch
> template files such as `*.tftpl`, which are treated as plain text.

## Prerequisites

- The calling job must grant `contents: write` so the fixes can be pushed (see
  [Usage](#usage)).
- The repository's Actions settings must allow workflows to write to the
  repository ("Read and write permissions" under Settings → Actions → General).

## Usage

Call it as a remote workflow from a workflow that runs on branch pushes, and
grant the job write access to contents:

```yaml
jobs:
  fmt:
    permissions:
      contents: write
    uses: codeforamerica/github-actions/.github/workflows/tofu-fmt.yaml@main
    with:
      path: ./
```

## Inputs

| Name   | Description                                   | Required | Default |
| ------ | --------------------------------------------- | -------- | ------- |
| `path` | The path to the files to format, recursively. | No       | `./`    |

## Outputs

_This workflow has no outputs._

[fmt]: https://opentofu.org/docs/cli/commands/fmt/
