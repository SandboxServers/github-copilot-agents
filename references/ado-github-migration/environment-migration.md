# Environment Migration

## ADO Environments → GitHub Environments

Both platforms have an "environment" concept for deployment targets, but the feature sets differ.

| Feature | ADO Environments | GitHub Environments |
|---|---|---|
| Approval gates | Pre/post deployment approvals | Required reviewers |
| Wait timer | Delay before deployment | Wait timer (up to 43,200 min) |
| Branch restrictions | Per-stage via conditions | Deployment branch policy |
| Deployment history | Tracks per environment | Tracks per environment |
| Resource targeting | VMs, Kubernetes, App Service | None (use actions for targeting) |
| Custom checks | Azure Function, REST, Azure Monitor | Custom deployment protection rules (via GitHub App) |

## Approval Gate Migration

### ADO Approvals → GitHub Required Reviewers

ADO:
```yaml
# ADO — approvals configured on environment in UI, referenced in pipeline
stages:
  - stage: DeployProd
    jobs:
      - deployment: deploy
        environment: production  # Has approval gate configured
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo deploying
```

GHA:
```yaml
# GHA — approvals configured on environment in Settings > Environments
jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment: production  # Has "Required reviewers" protection rule
    steps:
      - run: echo deploying
```

GitHub environment setup: Settings → Environments → production → Protection rules →
Required reviewers → Add up to 6 users/teams.

## Pre-Deployment Check Migration

### Required Reviewers — Direct Mapping

ADO approval gates → GitHub required reviewers. Supports up to 6 reviewers; only 1 needs to approve.

### Wait Timer — Direct Mapping

ADO wait timer → GitHub wait timer. Both delay deployment by a configured number of minutes.
GHA maximum is 43,200 minutes (30 days).

### Azure Function Check → Custom Deployment Protection Rule

ADO Azure Function checks call an HTTP endpoint and evaluate the response.
GHA equivalent requires a **GitHub App** implementing the deployment protection rule API.

```
ADO: Environment → Checks → Azure Function → polls HTTP endpoint
GHA: Environment → Protection rules → Custom → GitHub App webhook
```

The GitHub App receives a webhook when a deployment to the protected environment is requested,
performs validation, and calls back to approve or reject.

### REST API Check → Custom Deployment Protection Rule

Same pattern as Azure Function checks — implement as a GitHub App that receives the
`deployment_protection_rule` webhook event.

### Azure Monitor Check → Custom Protection Rule or Validation Job

Two migration approaches:

**Option 1: Custom deployment protection rule** (pre-deployment gate)
```
GitHub App receives webhook → queries Azure Monitor → approves/rejects
```

**Option 2: Separate validation job** (post-deployment check)
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - run: ./deploy.sh

  validate:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - run: |
          # Query Azure Monitor for health metrics
          FAILURES=$(az monitor metrics list \
            --resource /subscriptions/.../myapp \
            --metric Http5xx --interval PT5M \
            --query "value[0].timeseries[0].data[-1].total" -o tsv)
          if [ "$FAILURES" -gt "0" ]; then
            echo "Health check failed: $FAILURES 5xx errors"
            exit 1
          fi
```

### Business Hours Check → Custom Deployment Protection Rule

ADO business hours gate restricts deployments to configured windows.
GHA requires a custom deployment protection rule GitHub App, OR a simpler workflow condition:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    # Simple business hours check (UTC)
    if: >
      github.event_name == 'workflow_dispatch' ||
      (
        format('{0}', github.event.head_commit.timestamp) > '09:00' &&
        format('{0}', github.event.head_commit.timestamp) < '17:00'
      )
```

### Exclusive Lock → Concurrency Groups

ADO exclusive lock ensures only one deployment runs at a time per environment.

```yaml
# GHA — concurrency group prevents parallel deployments
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    concurrency:
      group: deploy-production
      cancel-in-progress: false  # Queue instead of cancel
    steps:
      - run: ./deploy.sh
```

`cancel-in-progress: false` queues new deployments (like ADO exclusive lock).
`cancel-in-progress: true` cancels the queued run (different behavior).

### Template Check → No Direct Equivalent

ADO template checks ensure pipelines use approved templates.
GHA alternatives:
- **Required workflows** (organization-level) — force specific workflows to run
- **CODEOWNERS** + branch protection — require review of workflow changes
- **Custom deployment protection rule** — validate workflow content before approving

## Deployment Strategy Migration

### runOnce → Direct Mapping

```yaml
# ADO runOnce
jobs:
  - deployment: deploy
    environment: production
    strategy:
      runOnce:
        deploy:
          steps:
            - script: ./deploy.sh

# GHA equivalent
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - run: ./deploy.sh
```

### rolling → Manual Implementation

ADO rolling strategy deploys to VM resources in batches. No GHA equivalent:

```yaml
# GHA — simulate rolling with matrix + max-parallel
jobs:
  deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        target: [vm-1, vm-2, vm-3, vm-4]
      max-parallel: 2  # Deploy to 2 at a time
    steps:
      - run: ./deploy.sh ${{ matrix.target }}
```

For Kubernetes rolling updates, use the infrastructure's native rolling capability:
```yaml
- uses: azure/k8s-deploy@v5
  with:
    manifests: deployment.yml
    strategy: basic  # Kubernetes handles rolling internally
```

### canary → Manual Implementation

ADO canary strategy with traffic splitting has no GHA equivalent.
Implement at the infrastructure level:

```yaml
jobs:
  canary:
    runs-on: ubuntu-latest
    environment: production-canary
    steps:
      - run: |
          # Deploy canary (e.g., Azure App Service slot)
          az webapp deployment slot create --name myapp --slot canary
          az webapp deploy --name myapp --slot canary --src-path app.zip

  validate-canary:
    needs: canary
    runs-on: ubuntu-latest
    steps:
      - run: ./smoke-tests.sh https://myapp-canary.azurewebsites.net

  promote:
    needs: validate-canary
    runs-on: ubuntu-latest
    environment: production  # Requires approval before swap
    steps:
      - run: az webapp deployment slot swap --name myapp --slot canary --target-slot production
```

## Deployment Job Lifecycle Hooks

ADO deployment jobs have lifecycle hooks: `preDeploy`, `deploy`, `routeTraffic`,
`postRouteTraffic`, `on.failure`, `on.success`. Map these to separate GHA jobs:

```yaml
jobs:
  pre-deploy:
    runs-on: ubuntu-latest
    steps:
      - run: ./pre-deploy-checks.sh

  deploy:
    needs: pre-deploy
    runs-on: ubuntu-latest
    environment: production
    steps:
      - run: ./deploy.sh

  post-deploy:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - run: ./smoke-tests.sh

  rollback:
    needs: deploy
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - run: ./rollback.sh
```

## Complete Example: 3-Stage Pipeline Migration

### ADO Original

```yaml
stages:
  - stage: Dev
    jobs:
      - deployment: deploy
        environment: dev
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: azure-dev
                    appName: myapp-dev

  - stage: Staging
    dependsOn: Dev
    condition: succeeded()
    jobs:
      - deployment: deploy
        environment: staging
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: azure-staging
                    appName: myapp-staging

  - stage: Production
    dependsOn: Staging
    condition: succeeded()
    jobs:
      - deployment: deploy
        environment: production  # Has approval gate
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: azure-prod
                    appName: myapp-prod
```

### GHA Migrated

```yaml
name: Deploy Pipeline
on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  deploy-dev:
    runs-on: ubuntu-latest
    environment: dev
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - uses: azure/webapps-deploy@v3
        with:
          app-name: myapp-dev

  deploy-staging:
    needs: deploy-dev
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - uses: azure/webapps-deploy@v3
        with:
          app-name: myapp-staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production  # Required reviewers configured here
    concurrency:
      group: deploy-production
      cancel-in-progress: false
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - uses: azure/webapps-deploy@v3
        with:
          app-name: myapp-prod
```

**Key migration notes for this example:**
- Each ADO stage becomes a GHA job with `needs:` for sequencing
- ADO `dependsOn` → GHA `needs:`
- ADO `condition: succeeded()` → GHA default behavior (jobs only run if dependencies succeed)
- ADO environment approvals → GHA environment required reviewers (configured in UI)
- ADO service connections → OIDC federated credentials (per-environment)
- ADO exclusive lock → GHA `concurrency` group with `cancel-in-progress: false`
