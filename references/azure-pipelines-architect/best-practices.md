# Azure Pipelines Best Practices

Proven patterns for building reliable, secure, and maintainable Azure Pipelines.

## Template Architecture

Maintain shared templates in a dedicated central repository:
- **Step templates** — individual build/test/deploy operations
- **Job templates** — complete CI or CD jobs with standard structure
- **Stage templates** — full deployment stages with environment gates
- **Extends templates** — enforce security policies across all pipelines

Reference the central repo via `resources.repositories` with a pinned `ref` (tag or branch):
```yaml
resources:
  repositories:
    - repository: templates
      type: git
      name: Platform/pipeline-templates
      ref: refs/tags/v2.1.0
```

Use `extends` templates to enforce organizational standards — required scanning steps, approved task lists, and mandatory compliance checks.

## Environment Protection Rules

Configure layered checks on each environment:
- **Approvals** — require one or more reviewers before deployment proceeds
- **Business hours** — restrict deployments to approved maintenance windows
- **Exclusive locks** — prevent concurrent deployments to the same environment
- **Branch control** — only allow deployments from `main` or `release/*` branches

```yaml
stages:
  - stage: Production
    jobs:
      - deployment: Deploy
        environment: production  # Checks enforced here
        strategy:
          runOnce:
            deploy:
              steps:
                - template: templates/steps/deploy-app.yml
```

## Secret Management via Key Vault

Link variable groups to Azure Key Vault for centralized secret management:
1. Create a variable group in Library → link to Key Vault
2. Select specific secrets to expose as pipeline variables
3. Reference in YAML via `- group: my-keyvault-vars`
4. Secrets are automatically masked in pipeline logs

Never store secrets as plain pipeline variables. Use workload identity federation for service connections to eliminate stored credentials entirely.

## Self-Hosted Agent Management

For self-hosted agents, design for reliability:
- **VMSS scale-set agents** — auto-scale based on queue depth, fresh image per run
- **Docker-based agents** — ephemeral containers with pre-built tool images
- **Maintenance jobs** — scheduled pipelines to prune Docker images, clear temp dirs, update SDKs
- **Agent pools per workload** — separate pools for build vs. deploy to isolate failures

Monitor agent health: disk usage, tool versions, and queue wait times via Azure DevOps Analytics.

## Dependency Caching

Use `Cache@2` to persist restored packages across builds:
```yaml
- task: Cache@2
  inputs:
    key: 'npm | "$(Agent.OS)" | package-lock.json'
    restoreKeys: |
      npm | "$(Agent.OS)"
    path: $(Pipeline.Workspace)/.npm
```

Key files by ecosystem:
| Ecosystem | Cache Key File | Cache Path |
|---|---|---|
| NuGet | `packages.lock.json` | `$(UserProfile)/.nuget/packages` |
| npm | `package-lock.json` | `$(Pipeline.Workspace)/.npm` |
| pip | `requirements.txt` | `$(Pipeline.Workspace)/.pip` |
| Maven | `pom.xml` | `$(Pipeline.Workspace)/.m2/repository` |

Caching typically saves 1-5 minutes per build depending on dependency count.

## Stage Dependencies and Deployment Order

Define explicit stage dependencies for clear deployment flow:
```yaml
stages:
  - stage: Build
  - stage: DeployDev
    dependsOn: Build
  - stage: DeployStaging
    dependsOn: DeployDev
  - stage: DeployProd
    dependsOn: DeployStaging
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
```

Use `condition:` on production stages to restrict which branches can deploy. Combine with environment approval gates for defense-in-depth.

## Security Hardening

Apply least-privilege principles throughout:
- **Service connections** — scope to specific resource groups, not subscriptions. Use workload identity federation
- **Pipeline permissions** — restrict who can edit pipeline YAML and manage service connections
- **Protected resources** — require approval checks on service connections, agent pools, variable groups, and environments
- **Extends templates** — enforce required steps (e.g., credential scanning) that pipeline authors can't remove
- **Fork build restrictions** — don't expose secrets to builds from forked repositories

Enable *Limit variables that can be set at queue time* and *Enable shell tasks arguments parameter validation* at the organization level.

## Artifact Management

Version and retain build artifacts systematically:
- Name artifacts with `$(Build.BuildId)` or semantic version for traceability
- Use pipeline artifacts (`PublishPipelineArtifact@1`) over classic build artifacts for better performance
- Set retention policies — keep production-deployed artifacts longer than PR artifacts
- Tag source commits on successful artifact publish for audit trails

```yaml
- task: PublishPipelineArtifact@1
  inputs:
    targetPath: $(Build.ArtifactStagingDirectory)
    artifact: drop-$(Build.BuildId)
```

## Pipeline Monitoring and Analytics

Track pipeline health metrics:
- **Duration trends** — identify regressions in build/deploy times via Pipeline Analytics
- **Failure rates** — monitor per-stage and per-task failure rates; set alerts for spikes
- **Agent utilization** — track queue wait times to right-size agent pools
- **Flaky test detection** — enable flaky test management to auto-quarantine unreliable tests

Use Azure DevOps Analytics views and Power BI dashboards for executive reporting. Set up Service Hook notifications for pipeline failures routed to Teams or Slack.

## Checklist Summary

| Practice | Priority | Impact |
|---|---|---|
| Central template repository | High | Consistency, reuse, governance |
| Environment protection rules | Critical | Controlled production deploys |
| Key Vault variable groups | Critical | Secure secret management |
| VMSS / Docker agents | High | Reliable, scalable builds |
| Dependency caching | Medium | Faster builds (1-5 min savings) |
| Explicit stage dependencies | High | Clear deployment order |
| Least-privilege connections | Critical | Reduced blast radius |
| Versioned artifacts + retention | High | Traceability and compliance |
| Pipeline analytics monitoring | Medium | Proactive issue detection |
