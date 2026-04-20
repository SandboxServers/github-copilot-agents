# Azure Pipelines Triggers and Resources

## CI Triggers

CI triggers run a pipeline when changes are pushed to matching branches/paths/tags.

```yaml
trigger:
  batch: true                     # Batch concurrent pushes into one run
  branches:
    include:
      - main
      - release/*
    exclude:
      - release/legacy-*
  paths:
    include:
      - src/**
      - build/**
    exclude:
      - src/docs/**
      - '**/*.md'
  tags:
    include:
      - v*
    exclude:
      - v*-beta
```

**Key behaviors:**
- `batch: true` — if a run is in progress and new pushes arrive, wait and batch them into one run
- Path filters are AND'd with branch filters (both must match)
- All trigger paths are **case-sensitive**
- The pipeline YAML version in the **pushed branch** determines CI trigger evaluation
- Wildcards: `*` matches one segment, `**` matches any number of segments

### Disable CI triggers
```yaml
trigger: none     # Manual or scheduled only — no CI trigger
```

## PR Triggers

PR triggers validate pull requests. In Azure Repos, YAML `pr:` triggers are supported; in GitHub repos, use branch policies or GitHub PR triggers.

```yaml
pr:
  autoCancel: true                # Cancel in-progress PR runs when new commits push
  drafts: false                   # Don't trigger on draft PRs
  branches:
    include:
      - main
      - release/*
  paths:
    include:
      - src/**
    exclude:
      - '**/*.md'
```

**Key behaviors:**
- `autoCancel: true` — previous PR validation runs are cancelled when new commits are pushed
- `drafts: false` — skip draft PRs (default is true — drafts DO trigger)
- The YAML version in the **source branch** of the PR determines PR trigger evaluation
- Path filters work the same as CI triggers

## Scheduled Triggers

Run pipelines on a cron schedule. Cron uses **UTC** timezone.

```yaml
schedules:
  - cron: '0 3 * * Mon-Fri'
    displayName: Weekday 3AM UTC build
    branches:
      include:
        - main
        - release/*
    always: false                  # Only run if there are source changes (default)

  - cron: '0 8 * * Sun'
    displayName: Weekly Sunday full build
    branches:
      include:
        - main
    always: true                   # Run even without changes — good for dependency scans
    batch: true                    # Don't start new run if previous scheduled run in progress
```

**Cron syntax:** `minute hour dayOfMonth month dayOfWeek`
- `0 3 * * Mon-Fri` = 3:00 AM UTC, Monday through Friday
- `0 */6 * * *` = every 6 hours
- `0 0 1 * *` = midnight on the 1st of each month

**Branch considerations:** The scheduled trigger definition in the **default branch** (usually `main`) is always evaluated. To add schedules for other branches, the cron definition must exist in the default branch YAML.

## Pipeline Triggers (Pipeline Completion)

Trigger one pipeline when another pipeline completes.

```yaml
resources:
  pipelines:
    - pipeline: buildPipeline             # Internal alias
      source: MyApp-CI                    # Name of the triggering pipeline
      project: MyProject                  # Required if in different project
      trigger:
        branches:
          include:
            - main
            - release/*
        stages:                           # Only trigger when these stages complete
          - Build
          - Test
        tags:                             # Only trigger on runs with these tags
          - production-ready

stages:
  - stage: Deploy
    jobs:
      - job: DeployJob
        steps:
          - download: buildPipeline       # Download artifacts from triggering pipeline
            artifact: drop
          - script: echo "Build number: $(resources.pipeline.buildPipeline.runName)"
```

## Repository Resources

Reference additional repositories for multi-repo pipelines or shared templates.

```yaml
resources:
  repositories:
    - repository: templates               # Alias used in template references
      type: git                           # Azure Repos
      name: SharedProject/pipeline-templates
      ref: refs/tags/v3.0                 # Pin to a tag

    - repository: configRepo
      type: github                        # GitHub repo
      endpoint: my-github-connection      # Service connection name
      name: myorg/config-repo
      ref: refs/heads/main
      trigger:                            # Trigger this pipeline on config changes
        branches:
          include:
            - main
        paths:
          include:
            - configs/**

steps:
  - checkout: self                        # Primary repo
  - checkout: configRepo                  # Additional repo checkout
  - template: stages/deploy.yml@templates # Use template from alias
```

## Container Resources

Define container images used as job containers or for sidecar services.

```yaml
resources:
  containers:
    - container: buildContainer
      image: mcr.microsoft.com/dotnet/sdk:8.0
      options: --cpus 2
    - container: testDb
      image: mcr.microsoft.com/mssql/server:2022-latest
      ports:
        - 1433:1433
      env:
        ACCEPT_EULA: Y
        SA_PASSWORD: $(dbPassword)

jobs:
  - job: Build
    container: buildContainer              # Run entire job inside container
    services:
      database: testDb                     # Sidecar container
    steps:
      - script: dotnet build
```

## Webhook Triggers

Trigger pipelines from external systems via incoming webhooks.

```yaml
resources:
  webhooks:
    - webhook: MyWebhookTrigger
      connection: MyIncomingWebhookSC      # Incoming webhook service connection
      filters:
        - path: repositoryName             # JSON path in payload
          value: maven-releases            # Expected value
        - path: action
          value: CREATED

steps:
  - script: |
      echo "Repository: ${{ parameters.MyWebhookTrigger.repositoryName }}"
      echo "Component: ${{ parameters.MyWebhookTrigger.component.group }}"
```

The external system POSTs JSON to the Azure DevOps webhook URL. The `filters` array ensures only matching payloads trigger the pipeline.

## Package Resources

Trigger on package updates from Azure Artifacts or other package feeds.

```yaml
resources:
  packages:
    - package: myNuGetPackage
      type: NuGet
      connection: myNuGetFeed
      name: MyCompany.SharedLib
      version: '>=2.0.0'                  # Semver range
      trigger: true                        # Trigger pipeline on new versions
```

## Common Trigger Issues

| Issue | Cause | Fix |
|---|---|---|
| CI trigger not firing | Pipeline YAML in pushed branch has `trigger: none` or different filter | Ensure trigger config exists in the branch being pushed |
| Scheduled trigger not running | Schedule defined only in feature branch, not in default branch | Schedule must be in default branch YAML |
| Path filter seems ignored | Path is case-sensitive or glob doesn't match | Verify exact casing; use `**` for recursive matching |
| Pipeline completion trigger not firing | Source pipeline name doesn't match | `source:` must match the pipeline **name** exactly (not filename) |
| PR trigger not firing in Azure Repos | Using `pr:` syntax but branch policies override it | Check if branch policies are configured (they take precedence) |
| `batch: true` not batching | Multiple pipeline definitions for same repo | Only works within a single pipeline definition |
| Multi-repo trigger fires but wrong repo checked out | Missing `checkout: self` | Always explicitly checkout repos you need |
