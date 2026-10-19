# Build Documentation with Zensical

The Build Documentation with Zensical workflow (`zensical-build.yaml`) builds
a documentation site with [Zensical], and uploads the built site as an
artifact. Pass the artifact to a deployment workflow, such as
[`shared-services-documentation-deploy.yaml`][documentation-deploy], to
publish it.

The workflow builds with `uv run --locked zensical build`, so Zensical, and any
plugins and extensions, are installed at the versions in your `uv.lock`. The
build fails if `uv.lock` is missing, or out of date with your `pyproject.toml`.

## Prerequisites

- A Zensical configuration file, such as `zensical.toml`, in your repository
- A `pyproject.toml` and `uv.lock` that include Zensical and any plugins or
  extensions in the same directory

If you don't have a `pyproject.toml` yet, you can create one, and add
Zensical, with [uv]:

```shell
uv init --bare
uv add zensical
```

Commit both `pyproject.toml` and `uv.lock`. When you add or update dependencies.
Use `uv add` or `uv lock` so the lockfile stays in sync.

## Usage

You can use the Build Documentation with Zensical workflow in your own workflow
by calling it as a remote workflow, and passing the artifact ID to the job
that deploys it.

```yaml
jobs:
  build:
    uses: codeforamerica/github-actions/.github/workflows/zensical-build.yaml@main

  deploy:
    needs: build
    uses: codeforamerica/github-actions/.github/workflows/shared-services-documentation-deploy.yaml@main
    with:
      artifact-ids: ${{ needs.build.outputs.artifact-id }}
      environment: ${{ inputs.environment || 'development' }}
    secrets: inherit
```

### Building from a subdirectory

If your documentation lives in a subdirectory of your repository, set
`working-directory` to the directory containing your Zensical configuration,
`pyproject.toml`, and `uv.lock`:

```yaml
    with:
      working-directory: docs-site
```

## Inputs

| Name                | Description                                                                                                   | Required | Default         |
| ------------------- | ------------------------------------------------------------------------------------------------------------- | -------- | --------------- |
| `working-directory` | Directory containing your Zensical configuration and `uv.lock`.                                               | No       | `.`             |
| `site-dir`          | Directory the site is built to, relative to `working-directory`. Must match `site_dir` in your configuration. | No       | `site`          |
| `artifact-name`     | Name of the artifact to upload the built site as.                                                             | No       | `documentation` |

## Outputs

| Name            | Description                                              |
| --------------- | -------------------------------------------------------- |
| `artifact-id`   | ID of the uploaded artifact containing the built site.   |
| `artifact-name` | Name of the uploaded artifact containing the built site. |

[documentation-deploy]: shared-services-documentation-deploy.md
[uv]: https://docs.astral.sh/uv/
[zensical]: https://zensical.org/
