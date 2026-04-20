# Azure Pipelines Performance Optimization

## Parallel Execution

Stages and jobs run in parallel by default unless `dependsOn` creates a dependency.

```yaml
# Two stages run in parallel (no dependency)
stages:
  - stage: UnitTests
    jobs:
      - job: Test
        steps:
          - script: dotnet test
  - stage: LintCheck
    jobs:
      - job: Lint
        steps:
          - script: npm run lint

# Sequential stages — Deploy depends on Build
stages:
  - stage: Build
    jobs:
      - job: Compile
        steps:
          - script: dotnet build
  - stage: Deploy
    dependsOn: Build         # Waits for Build to complete
```

## Matrix Strategy for Parallel Testing

Matrix duplicates a job with different variable combinations. Use `maxParallel` to cap concurrent jobs.

```yaml
jobs:
  - job: CrossPlatformTest
    strategy:
      maxParallel: 3
      matrix:
        linux:
          imageName: 'ubuntu-latest'
        mac:
          imageName: 'macos-latest'
        windows:
          imageName: 'windows-latest'
    pool:
      vmImage: $(imageName)
    steps:
      - script: dotnet test

# Parallel test slicing — split test suite across N agents
jobs:
  - job: ParallelTests
    strategy:
      parallel: 4            # 4 agents, each runs a slice
    steps:
      - task: VSTest@2
        inputs:
          testSelector: testAssemblies
          distributionBatchType: basedOnExecutionTime
```

## Conditional Execution

Skip work that isn't needed to save time and compute.

```yaml
stages:
  - stage: Build
    jobs:
      - job: BuildApp
        steps:
          - script: dotnet build

  - stage: DeployDev
    dependsOn: Build
    # Only deploy on main branch, not PR builds
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))

# Path-based conditions — skip backend deploy if only frontend changed
trigger:
  branches:
    include: ['main']
  paths:
    include: ['src/backend/**']    # This pipeline only triggers for backend changes
```

### Skip stages with runtime expressions

```yaml
parameters:
  - name: runTests
    type: boolean
    default: true

stages:
  - stage: Test
    condition: eq('${{ parameters.runTests }}', 'true')
    jobs:
      - job: UnitTests
        steps:
          - script: dotnet test
```

## Incremental Builds

Fetch only what changed to reduce checkout time:

```yaml
steps:
  - checkout: self
    fetchDepth: 1              # Shallow clone — only latest commit
    clean: false               # Preserve workspace between runs (self-hosted)
```

For monorepo patterns, combine shallow clone with path triggers so each pipeline only builds its component.

## Artifact Passing Optimization

- Use **pipeline artifacts** (not build artifacts) — dedup upload is 2-10x faster
- Minimize artifact size with `.artifactignore` files
- Only publish what downstream stages need — not entire build trees
- See [caching-artifacts.md](caching-artifacts.md) for detailed caching patterns

## Trigger Optimization

```yaml
trigger:
  batch: true                  # Coalesce multiple commits into one build
  branches:
    include: ['main', 'release/*']
  paths:
    exclude: ['docs/**', '*.md', '.github/**']  # Don't build for doc-only changes

pr:
  branches:
    include: ['main']
  paths:
    exclude: ['docs/**']
  drafts: false                # Don't build draft PRs
```

`batch: true` prevents queue explosion when developers push frequently — it waits for the current run to finish, then starts one run for all accumulated commits.

## Template Compilation Performance

Deeply nested template references slow pipeline compilation:

- **Limit nesting to 3-4 levels deep** — each level adds compilation time
- Avoid templates that include templates that include templates recursively
- `each` loops over large parameter lists expand at compile time — keep iteration counts reasonable
- Pre-expand templates using the REST API to debug slow compilation: `POST /_apis/pipelines/{id}/runs?api-version=7.1` with `previewRun: true`

## Agent Image Warm-Up

| Strategy | Startup Cost | Maintenance |
|----------|-------------|-------------|
| Microsoft-hosted (default image) | Tools pre-installed, ~0s for common tools | Zero |
| Microsoft-hosted + tool installer tasks | Download each run (30s-3min) | Low |
| Self-hosted with custom image | Pre-baked tools, ~0s | Image rebuild on updates |
| VMSS with custom image | Pre-baked + auto-scale | VMSS image management |

**Rule of thumb**: If a tool install takes >60 seconds per run and runs >10x/day, bake it into the agent image.

## Pipeline Analytics and Duration Budgets

Use built-in Pipeline Analytics (Pipelines > Analytics) to identify:

- **Longest-running tasks**: Target the top 3 for optimization
- **Queue wait time**: Indicates parallel job shortage — buy more or optimize scheduling
- **Pass rate trends**: Flaky tests waste re-run compute
- **P90 duration**: Set budget alerts when P90 exceeds your SLA

```yaml
# Add timing to custom scripts for manual analysis
steps:
  - script: |
      start=$(date +%s)
      dotnet build -c Release
      end=$(date +%s)
      echo "##vso[task.setvariable variable=buildSeconds]$((end-start))"
      echo "Build took $((end-start)) seconds"
    displayName: 'Build with timing'
```

## Quick Optimization Checklist

1. **Shallow clone** (`fetchDepth: 1`) — saves 10-60s on large repos
2. **Cache dependencies** — saves 30-180s for npm/NuGet/pip restores
3. **Path triggers** — avoids unnecessary builds entirely
4. **batch: true** — prevents queue flooding from rapid pushes
5. **Parallel stages** — run independent work concurrently
6. **Pipeline artifacts** — faster upload/download than build artifacts
7. **Skip draft PRs** — `drafts: false` on PR triggers
8. **Trim artifact size** — use `.artifactignore` aggressively
