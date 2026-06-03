# OpenTofu Changed Modules

The OpenTofu Changed Modules action (`opentofu-changed-modules/action.yaml`)
discovers all OpenTofu modules that have changed in a pull request or push to a
branch.

> [!TIP]
> Remember, OpenTofu configuration modules (or "layers") are just modules. Use
> this action to discover which configuration layer(s) have changed.

This can be used when you want to run another action only for the changed
modules, such as linting or planning the changed modules. A common use case is
to perform a plan on a lower-level environment, such as staging, on pull
requests.

## Usage

```yaml
jobs:
  my-job:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source code
        uses: actions/checkout@v6
      - name: Discover changed OpenTofu modules
        id: modules
        uses: codeforamerica/github-actions/.github/actions/opentofu-changed-modules@main
        with:
          # Optionally, set this if your modules are not at the default path.
          working-directory: ./tofu/configs
```

### Using with a matrix job

GitHub Actions allows you to run a job multiple times with different inputs
using a [matrix]. You can use the outputs of this action as the inputs for a
matrix job.

For example:

```yaml
jobs:
  changed-modules:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source code
        uses: actions/checkout@v6
      - name: Discover changed OpenTofu modules
        id: modules
        uses: codeforamerica/github-actions/.github/actions/opentofu-changed-modules@main
    outputs:
      modules: ${{ steps.modules.outputs.modules }}

  my-job:
    runs-on: ubuntu-latest
    needs: changed-modules
    strategy:
      matrix:
        module: ${{ fromJson(needs.changed-modules.outputs.modules) }}
    steps:
      - name: Checkout source code
        uses: actions/checkout@v6
      - name: Run a job for each changed module
        run: echo "Running a job for ${{ matrix.module }}"
      # ...
```


## Inputs

| Name                      | Description                                                                    | Required | Default        |
| ------------------------- | ------------------------------------------------------------------------------ | -------- | -------------- |
| `strip-working-directory` | When true, strips the working directory prefix from the returned module paths. | No       | `true`         |
| `working-directory`       | Path to the OpenTofu configurations.                                           | No       | `tofu/configs` |

## Outputs

| Name      | Description                                   |
| --------- | --------------------------------------------- |
| `modules` | A JSON array of all changed OpenTofu modules. |

[matrix]: https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/run-job-variations
