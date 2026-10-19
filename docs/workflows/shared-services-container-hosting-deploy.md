# Deploy to Shared Container Hosting

The Deploy to Shared Container Hosting workflow
(`shared-services-container-hosting-deploy.yaml`) deploys a new version of an
application hosted on Code for America's [shared services][shared-services]
container hosting platform. It triggers the deployment workflow in the
`shared-services-infra` repository, and waits for it to complete.

This workflow doesn't build or publish your application. Build and push your
images, and update the version parameter for each service, before calling
this workflow. The deployment uses whatever version the parameter points to for
your image tags.

## Prerequisites

- An application hosted on shared container hosting, with an app spec in the
  `shared-services-infra` repository
- A GitHub environment in your repository for each environment you deploy to,
  with the `DEPLOYMENT_APP_ID` and `DEPLOYMENT_APP_KEY` secrets set[^doppler]

## Usage

You can use the workflow in your own repository by calling it as a remote
workflow. Deployment credentials are environment secrets, so you must use
`secrets: inherit`. It runs in the environment you deploy to, and reads the
secrets from there.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    permissions:
      contents: read
      id-token: write
    steps:
      # Build and push your images, and update your version parameters...

  deploy:
    needs: build
    uses: codeforamerica/github-actions/.github/workflows/shared-services-container-hosting-deploy.yaml@main
    with:
      application: my-app
      environment: ${{ inputs.environment }}
    secrets: inherit
```

For more information, including a complete example, see the [user guide][guide]
for shared container hosting.

## Inputs

| Name                  | Description                                                                            | Required | Default       |
| --------------------- | -------------------------------------------------------------------------------------- | -------- | ------------- |
| `application`         | Name of the application's spec file in `shared-services-infra`, without the extension. | Yes      | n/a           |
| `environment`         | Environment to deploy to.                                                              | No       | `development` |
| `shared-services-ref` | Branch or tag of `shared-services-infra` to run the deployment from.                   | No       | `main`        |
| `timeout`             | Maximum time, in minutes, to wait for the deployment to complete.                      | No       | `30`          |

## Secrets

| Name                 | Description                                                 | Required |
| -------------------- | ----------------------------------------------------------- | -------- |
| `DEPLOYMENT_APP_ID`  | ID of the GitHub App used to trigger deployments.           | Yes      |
| `DEPLOYMENT_APP_KEY` | Private key for the GitHub App used to trigger deployments. | Yes      |

## Outputs

| Name      | Description                         |
| --------- | ----------------------------------- |
| `run-url` | URL of the deployment workflow run. |

[guide]: https://docs.dev.services.cfa.codes/shared-services/services/container-hosting/usage/
[shared-services]: https://docs.dev.services.cfa.codes/shared-services/

[^doppler]: These are managed by the DevOps team, and synced to your
  repository's environments from Doppler.
