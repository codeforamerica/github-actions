# Doppler OIDC Token

The Doppler OIDC Token action (`doppler-oidc-token/action.yaml`) retrieves a
single-use OIDC token for Doppler. This token can used to authenticate to
Doppler in a GitHub Actions workflow.

> [!IMPORTANT]
> If you're using this action to run OpenTofu commands, you _must_ create a new
> token in-between the `plan` and `apply` steps. This means that you can't do a
> plain `tofu apply` without providing a plan file.
>
> Use our OpenTofu plan and apply workflows to avoid this.

## Usage

```yaml
jobs:
  my-job:
    runs-on: ubuntu-latest
    steps:
      - name: Get an OIDC token for Doppler
        uses: codeforamerica/github-actions/.github/actions/doppler-oidc-token@main
```

### Configuration

The action itself has no inputs. You must set up a [Service Account
Identity][doppler-identity] ([runbook][doppler-identity-runbook]) in the GitHub
repository where this action is run.

## Inputs

_This action has no inputs._

## Outputs

_This action has no outputs._

The token is available to subsequent steps in the workflow as the
`DOPPLER_OIDC_TOKEN` environment variable. GitHub Actions automatically masks
this value in the logs to avoid leaking secrets.

[doppler-identity]: https://docs.doppler.com/docs/service-account-identities#configure-the-identity
[doppler-identity-runbook]: https://app.notion.com/p/cfa/Connect-a-Doppler-Service-Account-to-GitHub-38f373fd79b280fca599d3b3690f7e95
