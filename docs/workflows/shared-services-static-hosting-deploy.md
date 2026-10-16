# Deploy to Shared Static Hosting

The Deploy to Shared Static Hosting workflow
(`shared-services-static-hosting-deploy.yaml`) deploys a static application
to Code for America's [shared services][shared-services] static hosting
platform. It syncs your files to the shared S3 bucket, under your application's
prefix, then invalidates the CloudFront cache.

You can deploy files straight from your repository, or from an artifact
uploaded by an earlier build job.

## Prerequisites

- An application hosted on shared static hosting, with an app spec in the
  `shared-services-infra` repository
- A GitHub environment in your repository for each environment you deploy to,
  with the `DEPLOYMENT_APP_ID` and `DEPLOYMENT_APP_KEY` secrets set[^doppler]

## Usage

You can use the workflow in your own repository by calling it as a remote
workflow. The calling workflow must grant the `id-token: write` permission so
the workflow can assume your deployment role. The deployment role is an
environment secret, so you must use `secrets: inherit`.

```yaml
permissions:
  contents: read
  id-token: write

jobs:
  deploy:
    uses: codeforamerica/github-actions/.github/workflows/shared-services-static-hosting-deploy.yaml@main
    with:
      environment: ${{ inputs.environment || 'development' }}
    secrets: inherit
```

### Deploying a build artifact

If your site needs to be built first, upload the build output as an artifact
and pass its ID to the workflow:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      artifact-id: ${{ steps.upload.outputs.artifact-id }}
    steps:
      - uses: actions/checkout@v7
      - run: ./scripts/build
      - name: Upload build artifact
        id: upload
        uses: actions/upload-artifact@v7
        with:
          name: site
          path: dist/

  deploy:
    needs: build
    uses: codeforamerica/github-actions/.github/workflows/shared-services-static-hosting-deploy.yaml@main
    with:
      artifact_ids: ${{ needs.build.outputs.artifact-id }}
      environment: ${{ inputs.environment || 'development' }}
    secrets: inherit
```

## Inputs

| Name            | Description                                                                                       | Required | Default                  |
| --------------- | ------------------------------------------------------------------------------------------------- | -------- | ------------------------ |
| `environment`   | Environment to deploy to.                                                                         | No       | `development`            |
| `source_dir`    | Local directory to sync, relative to the repository or artifact root.                             | No       | `.`                      |
| `artifact_ids`  | Comma-separated list of artifact IDs to download. Takes precedence over `artifact_name`.          | No       | n/a                      |
| `artifact_name` | Name of a previously uploaded artifact to deploy. Prefer `artifact_ids` when the ID is available. | No       | n/a                      |
| `prefix`        | Prefix within the S3 bucket to sync files to. Should be the same as the application name.         | No       | `STATIC_PREFIX` variable |

## Secrets

| Name           | Description                                                | Required |
| -------------- | ---------------------------------------------------------- | -------- |
| `AWS_ROLE_ARN` | ARN of the IAM role authorized to deploy your application. | Yes      |

## Variables

These are read from the environment you deploy to.

| Name                         | Description                                                                    |
| ---------------------------- | ------------------------------------------------------------------------------ |
| `STATIC_BUCKET`              | S3 bucket where content is synced for the environment.                         |
| `STATIC_PREFIX`              | Prefix within the bucket to sync files to; should match your application name. |
| `CLOUDFRONT_DISTRIBUTION_ID` | ID of the CloudFront distribution serving the environment. Optional.           |
| `AWS_REGION`                 | AWS region of the shared services environment. Defaults to `us-east-1`.        |

[shared-services]: https://docs.dev.services.cfa.codes/shared-services/

[^doppler]: These are managed by the DevOps team, and synced to your
  repository's environments from Doppler.
