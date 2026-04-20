# Azure Pipelines Anti-Patterns

Common mistakes and anti-patterns that degrade pipeline reliability, security, and maintainability.

## Monolithic Pipeline YAML

Stuffing every stage, job, and step into a single `azure-pipelines.yml` file. Results in 1000+ line files that are impossible to review, test independently, or reuse across projects.

**Fix:** Decompose into stage, job, and step templates. Keep the root YAML as an orchestrator that references templates.

## No Template Reuse (Copy-Paste Pipelines)

Duplicating pipeline YAML across repositories instead of referencing shared templates from a central repo. When a build task needs updating, you touch dozens of files.

**Fix:** Publish shared templates in a dedicated `pipeline-templates` repository. Reference via `resources.repositories` with a pinned `ref`.

## Secrets in Plain Pipeline Variables

Defining secrets directly in pipeline YAML variables or as plain-text pipeline variables in the UI. These are visible in logs, editable by contributors, and not auditable.

**Fix:** Store secrets in Azure Key Vault and link via variable groups. Use `$(secret-name)` syntax — values are automatically masked in logs.

## No Environment Gates

Deploying directly to production without approval gates, business-hours checks, or exclusive locks. A broken merge triggers an unattended production deployment.

**Fix:** Configure environment checks: approvals, business hours, branch control, and exclusive locks. Use `environment:` in deployment jobs to enforce gates.

## Overly Broad Triggers

```yaml
# Anti-pattern: triggers on every file change
trigger:
  branches:
    include:
      - '*'
```

This fires CI on documentation edits, README changes, and .gitignore updates — wasting agent time and parallel job capacity.

**Fix:** Use path filters to scope triggers to source code directories:
```yaml
trigger:
  branches:
    include: [main]
  paths:
    include: [src/*, tests/*]
    exclude: [docs/*, '*.md']
```

## Unmaintained Self-Hosted Agents

Self-hosted agents deployed once and never updated. Over time: outdated SDKs, full disks, expired certificates, stale Docker images. Builds fail with cryptic errors.

**Fix:** Use VMSS scale-set agents with fresh images, or run ephemeral Docker-based agents. Schedule maintenance jobs to prune Docker, update tools, and clear temp files.

## No Artifact Versioning

Publishing build outputs without version stamps. You can't trace a deployed artifact back to a specific commit, branch, or build run.

**Fix:** Use `$(Build.BuildId)` or semantic versioning in artifact names. Tag the source commit on successful publish. Leverage `BuildNumber` format with date and revision.

## Inline Scripts Instead of Task References

Embedding complex bash/PowerShell scripts directly in YAML steps instead of using versioned task references or script files checked into the repo.

**Fix:** Move scripts to `scripts/` in the repo and call them via `- script: ./scripts/deploy.sh`. For common operations, use built-in tasks (`DotNetCoreCLI@2`, `AzureCLI@2`).

## No Caching

Restoring NuGet packages, npm modules, or pip dependencies from scratch on every build. Adds 2-5 minutes per run and increases network load.

**Fix:** Use the `Cache@2` task with appropriate key files (`packages.lock.json`, `package-lock.json`, `requirements.txt`).

## Using Classic Pipelines When YAML Is Available

Creating new pipelines with the classic (GUI) editor. Classic pipelines can't be code-reviewed, versioned in Git, or templated across projects.

**Fix:** Use YAML pipelines for all new work. Migrate existing classic pipelines incrementally — export the classic definition as a reference, then rewrite in YAML.

## No Retry Logic for Flaky Steps

Deployment steps that call external APIs (Azure, Kubernetes, package feeds) fail intermittently due to transient errors. Without retries, the entire pipeline fails.

**Fix:** Use `retryCountOnTaskFailure` on tasks prone to transient failures:
```yaml
- task: AzureWebApp@1
  retryCountOnTaskFailure: 2
  inputs:
    azureSubscription: $(serviceConnection)
    appName: $(appName)
```

## Pipeline as Root (Over-Privileged Service Connections)

Granting service connections Owner or Contributor access to the entire subscription. A compromised pipeline can delete resources, exfiltrate data, or escalate privileges.

**Fix:** Scope service connections to specific resource groups. Use workload identity federation instead of secrets. Apply RBAC with least-privilege roles (e.g., Website Contributor for App Service deployments).

## Summary Checklist

| Anti-Pattern | Risk | Severity |
|---|---|---|
| Monolithic YAML | Unmaintainable, no reuse | High |
| Copy-paste pipelines | Drift, inconsistent builds | High |
| Plain-text secrets | Credential exposure | Critical |
| No environment gates | Uncontrolled production deploys | Critical |
| Broad triggers | Wasted compute, slow feedback | Medium |
| Stale self-hosted agents | Random build failures | High |
| No artifact versioning | Can't trace deployments | Medium |
| Inline scripts | Hard to maintain and test | Medium |
| No caching | Slow builds | Medium |
| Classic pipelines | No GitOps, no review | Medium |
| No retry logic | Flaky failures block releases | Medium |
| Over-privileged connections | Security breach risk | Critical |
