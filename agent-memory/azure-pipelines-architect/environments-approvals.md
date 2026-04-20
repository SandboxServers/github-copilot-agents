# Azure Pipelines Environments, Deployment Strategies, and Approvals

## Environment Definition

Environments represent deployment targets (dev, staging, production) and provide traceability, deployment history, and approval gates.

Environments are created automatically when referenced in a `deployment` job, or manually via the Azure DevOps UI under Pipelines > Environments.

```yaml
# Environment is created automatically on first reference
stages:
  - stage: Deploy
    jobs:
      - deployment: DeployWeb
        environment: production             # Creates 'production' env if it doesn't exist
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo "Deploying to production"
```

**Environment naming with resource targets:**
```yaml
environment: production.my-k8s-namespace    # Environment 'production', resource 'my-k8s-namespace'
```

## Deployment Strategies

Deployment jobs (`deployment:` instead of `job:`) support three strategies: `runOnce`, `rolling`, and `canary`.

### runOnce Strategy

The simplest strategy — deploy once, verify, complete.

```yaml
jobs:
  - deployment: DeployWeb
    displayName: Deploy Web App
    environment: staging
    pool:
      vmImage: ubuntu-latest
    strategy:
      runOnce:
        preDeploy:
          steps:
            - script: echo "Validating prerequisites"
            - task: AzureCLI@2
              inputs:
                azureSubscription: $(serviceConnection)
                scriptType: bash
                inlineScript: az webapp show --name $(appName) --query state
        deploy:
          steps:
            - download: current
              artifact: drop
            - task: AzureWebApp@1
              inputs:
                azureSubscription: $(serviceConnection)
                appName: $(appName)
                package: $(Pipeline.Workspace)/drop/*.zip
        routeTraffic:
          steps:
            - script: echo "Routing traffic to new deployment"
        postRouteTraffic:
          steps:
            - script: echo "Running smoke tests"
            - task: AzureCLI@2
              inputs:
                azureSubscription: $(serviceConnection)
                scriptType: bash
                inlineScript: |
                  curl -f https://$(appName).azurewebsites.net/health || exit 1
        on:
          failure:
            steps:
              - script: echo "Deployment failed — initiating rollback"
          success:
            steps:
              - script: echo "Deployment succeeded"
```

### Lifecycle Hooks (All Strategies)

| Hook | When It Runs | Typical Use |
|---|---|---|
| `preDeploy` | Before deployment starts | Validate prerequisites, take backups |
| `deploy` | Main deployment | Deploy artifacts, run migrations |
| `routeTraffic` | After deploy succeeds | Switch traffic, update load balancer |
| `postRouteTraffic` | After traffic routing | Smoke tests, health checks |
| `on.failure` | If any hook fails | Rollback, send alerts |
| `on.success` | If all hooks succeed | Cleanup, notifications |

### Rolling Strategy

Deploy incrementally across a set of VMs or targets. Used with environment VM resources.

```yaml
jobs:
  - deployment: DeployToVMs
    environment:
      name: production
      resourceType: VirtualMachine
      tags: web                             # Target only VMs tagged 'web'
    strategy:
      rolling:
        maxParallel: 2                      # Deploy to 2 VMs at a time
        # Can also be a percentage: maxParallel: 25%
        preDeploy:
          steps:
            - script: echo "Pre-deploy on $(Agent.MachineName)"
        deploy:
          steps:
            - task: IISWebAppDeploymentOnMachineGroup@0
              inputs:
                WebSiteName: MyWebSite
                Package: $(Pipeline.Workspace)/drop/*.zip
        on:
          failure:
            steps:
              - script: echo "Failed on $(Agent.MachineName)"
```

### Canary Strategy

Incrementally roll out to a percentage of targets, then expand.

```yaml
jobs:
  - deployment: DeployCanary
    environment:
      name: production
      resourceType: VirtualMachine
    strategy:
      canary:
        increments: [10, 50]                # First 10%, then 50%, then 100%
        preDeploy:
          steps:
            - script: echo "Pre-deploy canary"
        deploy:
          steps:
            - script: echo "Deploying to $(Strategy.CycleName) — increment $(Strategy.Increment)"
        routeTraffic:
          steps:
            - script: echo "Routing $(Strategy.Increment)% traffic"
        postRouteTraffic:
          steps:
            - script: echo "Monitoring canary health at $(Strategy.Increment)%"
        on:
          failure:
            steps:
              - script: echo "Canary failed at $(Strategy.Increment)% — rolling back"
```

**Canary variables:** `$(Strategy.CycleName)` is `baseline`, `canary`, or `primary`. `$(Strategy.Increment)` is the current percentage.

## Environment Approvals and Checks

Configure approvals and checks on an environment via Azure DevOps UI: Pipelines > Environments > (select environment) > Approvals and checks.

### Approval Types

**Manual approval:**
- One or more required approvers (users or groups)
- Timeout (default: 30 days)
- Approval policies: require a minimum number of approvers, allow the requestor to approve
- Instructions shown to approvers

**Business hours check:**
- Only allow deployments during specified time windows
- Configured timezone, day-of-week, and hour ranges

### Check Types

| Check Type | Purpose | Configuration |
|---|---|---|
| **Approvals** | Human gating | Specify approvers, policies, timeout |
| **Branch control** | Restrict which branches can deploy | Allowed branches pattern |
| **Business hours** | Time-window restriction | Timezone, days, hours |
| **Azure Function** | Custom validation logic | Function URL, key, success criteria |
| **REST API** | Call external system for approval | URL, method, headers, success criteria |
| **Azure Monitor** | Alert-based gating | Monitor alert rules, evaluation window |
| **Template check** | Require pipeline extends a specific template | Template path and repository |
| **Exclusive lock** | Serialize deployments to environment | Only one run accesses environment at a time |

### Exclusive Lock

Prevents concurrent deployments to the same environment — essential for production.

When enabled, if a pipeline run is deploying to the environment and another run reaches that stage, it waits in queue. The lock mode can be:
- **Sequential** — queued runs deploy in order
- **Latest only** — only the most recent queued run deploys (others are cancelled)

## Multi-Environment Deployment Pattern

```yaml
stages:
  - stage: Build
    jobs:
      - job: BuildApp
        pool:
          vmImage: ubuntu-latest
        steps:
          - script: dotnet publish -c Release -o $(Build.ArtifactStagingDirectory)
          - publish: $(Build.ArtifactStagingDirectory)
            artifact: drop

  - stage: DeployDev
    dependsOn: Build
    jobs:
      - deployment: Deploy
        environment: dev                    # No approvals — auto-deploys
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  artifact: drop
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: sc-dev
                    appName: myapp-dev
                    package: $(Pipeline.Workspace)/drop/*.zip

  - stage: DeployStaging
    dependsOn: DeployDev
    jobs:
      - deployment: Deploy
        environment: staging                # Approval check configured in UI
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  artifact: drop
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: sc-staging
                    appName: myapp-staging
                    package: $(Pipeline.Workspace)/drop/*.zip

  - stage: DeployProd
    dependsOn: DeployStaging
    jobs:
      - deployment: Deploy
        environment: production             # Approval + exclusive lock + business hours
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  artifact: drop
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: sc-prod
                    appName: myapp-prod
                    package: $(Pipeline.Workspace)/drop/*.zip
```

## Environment vs Stage-Level Controls

| Feature | Environment Checks | Stage Conditions |
|---|---|---|
| Human approvals | Yes — approvers, policies | No |
| Branch restrictions | Yes — branch control check | Yes — via `condition:` expression |
| Time windows | Yes — business hours check | No |
| Exclusive locks | Yes — serialized deployments | No |
| Custom logic | Yes — Azure Function / REST API | Yes — via output variables in conditions |
| Deployment history | Yes — tracked per environment | No |
| Kubernetes/VM targeting | Yes — environment resources | No |

**Best practice:** Use **environment checks** for governance (approvals, compliance, business hours). Use **stage conditions** for technical flow control (skip stage if tests fail, conditional deployments based on branch).

## Service Connections and Environments

Service connections are NOT directly tied to environments, but they often pair together. A deployment job references an environment for governance and a service connection for credentials:

```yaml
- deployment: Deploy
  environment: production                   # Governance: approvals, checks, history
  pool:
    vmImage: ubuntu-latest
  strategy:
    runOnce:
      deploy:
        steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: sc-production   # Credentials: service connection
              appName: myapp-prod
```

**Lock down service connections** by restricting their pipeline authorization — only specific pipelines should access production service connections.
