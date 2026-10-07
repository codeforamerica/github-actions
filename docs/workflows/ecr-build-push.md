# Build and Push Image to ECR

The Build and Push Image to ECR workflow (`ecr-build-push.yaml`) builds a Docker
image with [Buildx] and pushes it to an Amazon ECR repository. By default, the
image is tagged with the commit SHA.

[Docker Hardened Images][dhi] (DHI) are our baseline for base images, so the
workflow logs in to `dhi.io` before building unless you opt out.

Provenance and SBOM attestations are disabled by default. Attestations cause
Buildx to push an image index rather than a single manifest, and Amazon
Inspector reports image indexes as `UnsupportedImageError`, leaving findings on
an untagged child manifest.

## Prerequisites

- An ECR repository in the target AWS account
- An IAM role with permission to push to the repository, and an OIDC connection
  between your GitHub repository and the target AWS account[^keys]
- Docker Hub credentials with read access to DHI, unless you opt out of the
  registry login
- A `Dockerfile` in your repository

## Usage

You can use the Build and Push Image to ECR workflow in your own workflow by
calling it as a remote workflow. The calling job must grant the `id-token:
write` permission so the workflow can assume the IAM role.

```yaml
jobs:
  build:
    uses: codeforamerica/github-actions/.github/workflows/ecr-build-push.yaml@main
    permissions:
      contents: read
      id-token: write
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      REGISTRY_USERNAME: ${{ secrets.DOCKER_HUB_USERNAME }}
      REGISTRY_PASSWORD: ${{ secrets.DOCKER_HUB_READONLY_TOKEN }}
    with:
      ecr-repository: my-app-staging-web

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying ${{ needs.build.outputs.image }}"
```

### Specifying the AWS account

The workflow pushes to `<account>.dkr.ecr.<region>.amazonaws.com/<ecr-repository>`.
By default, `<account>` is the account of the role or access keys you pass, and
`<region>` is the `aws-region` input. If the repository is in a different
account than your credentials, set `aws-account-id` explicitly. The credentials
must have permission to push to that repository.

```yaml
    with:
      ecr-repository: my-app-staging-web
      aws-account-id: "123456789012"
```

### Inheriting secrets

If your repository has access to the org-wide `DOCKER_HUB_USERNAME` and
`DOCKER_HUB_READONLY_TOKEN` secrets, you can use `secrets: inherit` instead of
passing them. The workflow falls back to those names for the registry login.

`secrets: inherit` passes all of your secrets by name and can't be combined
with explicit mappings. Your other secrets must use the names this workflow
expects, such as `AWS_ROLE_ARN` and `BUILD_SECRETS`.

```yaml
jobs:
  build:
    uses: codeforamerica/github-actions/.github/workflows/ecr-build-push.yaml@main
    permissions:
      contents: read
      id-token: write
    secrets: inherit
    with:
      ecr-repository: my-app-staging-web
```

### Using a different registry

To pull base images from a different private registry, set the `registry` input
and pass credentials for that registry instead. If your base images are public
and you don't need a registry login, set `registry` to an empty string:

```yaml
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
    with:
      ecr-repository: my-app-staging-web
      registry: ""
```

### Build secrets

Pass [build secrets][build-secrets] as one `id=value` pair per line. They're
available to `RUN --mount=type=secret,id=<id>` instructions in your Dockerfile.

```yaml
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
      BUILD_SECRETS: |
        github_pat=${{ secrets.DEPLOY_PAT }}
```

## Inputs

| Name              | Description                                                         | Required | Default                     |
| ----------------- | ------------------------------------------------------------------- | -------- | --------------------------- |
| `ecr-repository`  | Name of the ECR repository to push to.                              | Yes      | n/a                         |
| `aws-account-id`  | AWS account that owns the repository.                               | No       | Account of the credentials  |
| `image-tag`       | Primary tag for the image.                                          | No       | `${{ github.sha }}`         |
| `additional-tags` | Additional tags, separated by whitespace or newlines.               | No       | n/a                         |
| `context`         | Docker build context.                                               | No       | `.`                         |
| `file`            | Path to the Dockerfile.                                             | No       | `Dockerfile` in the context |
| `target`          | Build stage to target.                                              | No       | n/a                         |
| `build-args`      | Build arguments, one `KEY=value` per line.                          | No       | n/a                         |
| `platforms`       | Target platforms for the build.                                     | No       | `linux/amd64`               |
| `provenance`      | Generate a provenance attestation.                                  | No       | `false`                     |
| `sbom`            | Generate an SBOM attestation.                                       | No       | `false`                     |
| `registry`        | Registry to log in to before building. Empty string to skip.        | No       | `dhi.io`                    |
| `aws-region`      | AWS region of the ECR repository.                                   | No       | `us-east-1`                 |
| `environment`     | GitHub environment to run in, if your secrets are scoped to one.    | No       | n/a                         |

## Secrets

| Name                    | Description                                            | Required |
| ----------------------- | ------------------------------------------------------ | -------- |
| `AWS_ROLE_ARN`          | ARN of the IAM role to assume.                         | No[^aws] |
| `AWS_ACCESS_KEY_ID`     | AWS access key ID, if not using `AWS_ROLE_ARN`.        | No[^aws] |
| `AWS_SECRET_ACCESS_KEY` | AWS secret access key, if not using `AWS_ROLE_ARN`.    | No[^aws] |
| `REGISTRY_USERNAME`     | Username for the registry set in the `registry` input. | No[^reg] |
| `REGISTRY_PASSWORD`     | Password or token for the registry set in `registry`.  | No[^reg] |
| `BUILD_SECRETS`         | Build secrets, one `id=value` per line.                | No       |

## Outputs

| Name     | Description                                                          |
| -------- | -------------------------------------------------------------------- |
| `image`  | Full image URI with the primary tag, e.g. `<registry>/<repo>:<sha>`. |
| `digest` | Digest of the pushed image.                                          |

[build-secrets]: https://docs.docker.com/build/building/secrets/
[buildx]: https://docs.docker.com/build/buildx/
[dhi]: https://docs.docker.com/dhi/

[^aws]: Either `AWS_ROLE_ARN` or both `AWS_ACCESS_KEY_ID` and
  `AWS_SECRET_ACCESS_KEY` must be provided.

[^reg]: Required unless `registry` is set to an empty string, or secrets are
  inherited and `DOCKER_HUB_USERNAME` and `DOCKER_HUB_READONLY_TOKEN` are set.

[^keys]: Static access keys are supported for repositories that haven't moved
  to OIDC yet, but an IAM role is preferred.
