# Azure Pipelines Caching and Artifact Management

## Pipeline Caching with Cache@2

The Cache@2 task stores and restores files between pipeline runs. Caches are **immutable** — once created, contents cannot be modified. The task works in two phases: restore at step execution, save in an automatic post-job step.

```yaml
steps:
  - task: Cache@2
    displayName: Cache NuGet packages
    inputs:
      key: 'nuget | "$(Agent.OS)" | $(Build.SourcesDirectory)/**/packages.lock.json'
      restoreKeys: |
        nuget | "$(Agent.OS)"
        nuget
      path: $(NUGET_PACKAGES)
      cacheHitVar: CACHE_RESTORED
  - script: dotnet restore --no-cache
    condition: ne(variables.CACHE_RESTORED, 'true')  # Skip restore on cache hit
```

### Key Design Principles

Cache keys use `|` as segment separators. File path segments are hashed automatically. Design keys from **broad to specific** so `restoreKeys` can match partial prefixes.

- `cacheHitVar` is set to `true` (exact hit), `inexact` (restoreKey match), or `false` (miss)
- Caches are scoped to **branch** by default — a cache created on `feature/x` is not visible to `main`
- **Cross-branch restore**: `restoreKeys` can match caches from the default branch as fallback
- Caches are immutable: same key = same content, no overwrites

## Language-Specific Cache Paths

```yaml
# Node.js / npm
variables:
  npm_config_cache: $(Pipeline.Workspace)/.npm
steps:
  - task: Cache@2
    inputs:
      key: 'npm | "$(Agent.OS)" | package-lock.json'
      path: $(npm_config_cache)

# Python / pip
variables:
  PIP_CACHE_DIR: $(Pipeline.Workspace)/.pip
steps:
  - task: Cache@2
    inputs:
      key: 'pip | "$(Agent.OS)" | requirements.txt'
      path: $(PIP_CACHE_DIR)

# Maven
variables:
  MAVEN_CACHE_FOLDER: $(Pipeline.Workspace)/.m2/repository
steps:
  - task: Cache@2
    inputs:
      key: 'maven | "$(Agent.OS)" | **/pom.xml'
      path: $(MAVEN_CACHE_FOLDER)

# Go modules
variables:
  GOMODCACHE: $(Pipeline.Workspace)/.go/mod
steps:
  - task: Cache@2
    inputs:
      key: 'go | "$(Agent.OS)" | go.sum'
      path: $(GOMODCACHE)
```

## Pipeline Artifacts (Modern)

Pipeline artifacts replace legacy build artifacts. They are free from storage billing and use dedup-based upload for speed.

```yaml
# Publish — shorthand syntax
steps:
  - publish: $(Build.ArtifactStagingDirectory)/output
    artifact: MyApp

# Publish — full task syntax
  - task: PublishPipelineArtifact@1
    inputs:
      targetPath: $(Build.ArtifactStagingDirectory)/output
      artifactName: MyApp

# Download in a later stage
stages:
  - stage: Deploy
    jobs:
      - job: DeployWeb
        steps:
          - download: current        # 'current' = same pipeline run
            artifact: MyApp
          - task: DownloadPipelineArtifact@2
            inputs:
              source: specific       # Cross-pipeline download
              project: MyProject
              pipeline: 42
              runVersion: latest
              artifact: MyApp
              path: $(Pipeline.Workspace)/external
```

### .artifactignore

Place `.artifactignore` in the targetPath directory (syntax mirrors `.gitignore`):

```
**/*
!*.dll
!*.exe
!*.config
```

## Build Artifacts (Legacy) vs Pipeline Artifacts

| Feature | Build Artifacts (PublishBuildArtifacts) | Pipeline Artifacts (PublishPipelineArtifact) |
|---------|---------------------------------------|---------------------------------------------|
| Upload speed | Single-threaded | Dedup-based, parallel upload |
| Storage billing | Counts toward storage | Free |
| Cross-stage | Requires explicit download task | `download: current` shorthand |
| Recommended | No — legacy | Yes — modern default |

## Universal Packages via Azure Artifacts

For versioned artifact distribution beyond a single pipeline, publish to Azure Artifacts feeds:

```yaml
- task: UniversalPackages@0
  inputs:
    command: publish
    feedsToUse: internal
    vstsFeed: MyFeed
    vstsFeedPackage: my-app
    versionOption: patch        # auto-increment patch version
    packagePublishDescription: 'Build $(Build.BuildNumber)'
```

## Artifact Retention

- Pipeline artifacts are deleted when the pipeline run is deleted
- Default retention: 30 days for non-release runs (configurable in project settings)
- Artifacts in release pipelines can be retained indefinitely via retention policies
- **Caution**: Deleting a pipeline run deletes ALL associated artifacts

## Common Anti-Patterns

- **Caching too much**: Caching entire `node_modules` instead of the npm cache directory — slower than a fresh install
- **Wrong cache key**: Using `**/*.js` as a hash input (changes every build, never hits cache)
- **Caching build output**: Use artifacts for build output — caching is for dependencies that rarely change
- **No restoreKeys**: Missing fallback keys means a cache miss on any dependency change
- **Cross-stage caching**: Cache is job-scoped — use artifacts to pass files between stages/jobs
