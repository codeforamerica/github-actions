# Deploy to Shared Documentation Hosting

The Deploy to Shared Documentation Hosting workflow
(`shared-services-documentation-deploy.yaml`) deploys documentation to Code
for America's [shared services][shared-services] documentation platform. It
syncs your built documentation to the shared S3 bucket, under your
application's prefix. Files that no longer exist in your build are removed.

The workflow doesn't build your documentation. Build it in an earlier job, and
upload the built site as an artifact. If you're using Zensical, the
[`zensical-build.yaml`][zensical-build] workflow does this for you. For other
documentation tools, use your own build job and upload the output with
`actions/upload-artifact`.

## Prerequisites

- Documentation configured for the shared services documentation platform, with
  an app spec in the `shared-services-infra` repository
- A GitHub environment in your repository for each environment you deploy to,
  with the secrets and variables below set[^doppler]

## Usage

You can use the Deploy to Shared Documentation Hosting workflow in your own
workflow by calling it as a remote workflow. The calling workflow must grant
the `id-token: write` permission so the workflow can assume your deployment
role.

> [!TIP]
> By specifying `${{ inputs.environment || 'development' }}`, you can set the
> default environment to 'development'. This is useful for workflows that can be
> triggered in a context where `inputs` is unavailable.

```yaml
permissions:
  contents: read
  id-token: write

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

The deployment role comes from your GitHub environment's secret, so you must
explicitly give the remote workflow access to it. You can do this by setting
`secrets: inherit`, as demonstrated above, or by explicitly specifying the
secret in your workflow:

> [!TIP]
> If your secrets don't match the expected names in the remote workflow, you
> _must_ explicitly map them as shown below.

```yaml
  deploy:
    needs: build
    uses: codeforamerica/github-actions/.github/workflows/shared-services-documentation-deploy.yaml@main
    with:
      artifact-ids: ${{ needs.build.outputs.artifact-id }}
      environment: ${{ inputs.environment || 'development' }}
    secrets:
      AWS_DOCS_ROLE_ARN: ${{ secrets.AWS_DOCS_ROLE_ARN }}
```

### Using a different documentation tool

If you build your documentation with another tool, replace the `build` job with
your own, and upload the built site as an artifact:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      artifact-id: ${{ steps.upload.outputs.artifact-id }}
    steps:
      - uses: actions/checkout@v7
      - run: make build
      - name: Upload built site
        id: upload
        uses: actions/upload-artifact@v7
        with:
          name: documentation
          path: site/

  deploy:
    needs: build
    uses: codeforamerica/github-actions/.github/workflows/shared-services-documentation-deploy.yaml@main
    with:
      artifact-ids: ${{ needs.build.outputs.artifact-id }}
      environment: ${{ inputs.environment || 'development' }}
    secrets: inherit
```

## Inputs

| Name            | Description                                                                                       | Required | Default                |
| --------------- | ------------------------------------------------------------------------------------------------- | -------- | ---------------------- |
| `environment`   | Environment to deploy to.                                                                         | No       | `development`          |
| `artifact-ids`  | Comma-separated list of artifact IDs to download. Takes precedence over `artifact-name`.          | No       | n/a                    |
| `artifact-name` | Name of a previously uploaded artifact to deploy. Prefer `artifact-ids` when the ID is available. | No       | n/a                    |
| `source-dir`    | Local directory to sync, relative to the repository or artifact root.                             | No       | `.`                    |
| `prefix`        | Prefix within the S3 bucket to sync files to. Should be the same as the application name.         | No       | `DOCS_PREFIX` variable |

## Secrets

| Name                | Description                                                  | Required |
| ------------------- | ------------------------------------------------------------ | -------- |
| `AWS_DOCS_ROLE_ARN` | ARN of the IAM role authorized to deploy your documentation. | Yes      |

## Variables

These are read from the environment you deploy to.

| Name          | Description                                                                    |
| ------------- | ------------------------------------------------------------------------------ |
| `DOCS_BUCKET` | S3 bucket where documentation is synced for the environment.                   |
| `DOCS_PREFIX` | Prefix within the bucket to sync files to; should match your application name. |
| `AWS_REGION`  | AWS region of the shared services environment. Defaults to `us-east-1`.        |

[shared-services]: https://github.com/codeforamerica/shared-services-infra
[zensical-build]: zensical-build.md

[^doppler]: These are managed by the DevOps team, and synced to your
  repository's environments from Doppler.
