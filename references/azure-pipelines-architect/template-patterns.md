# Azure Pipelines Template Composition Patterns

## Template Types

Azure Pipelines supports four template insertion points — **stage**, **job**, **step**, and **variable** templates — plus **extends** templates that wrap the entire pipeline.

### Step Template
```yaml
# templates/steps/dotnet-build.yml
parameters:
  - name: project
    type: string
  - name: configuration
    type: string
    default: Release

steps:
  - task: DotNetCoreCLI@2
    displayName: Build ${{ parameters.project }}
    inputs:
      command: build
      projects: ${{ parameters.project }}
      arguments: '--configuration ${{ parameters.configuration }}'
```

### Job Template
```yaml
# templates/jobs/test-job.yml
parameters:
  - name: testProjects
    type: string
  - name: pool
    type: object
    default:
      vmImage: ubuntu-latest

jobs:
  - job: Test
    pool: ${{ parameters.pool }}
    steps:
      - template: ../steps/dotnet-build.yml
        parameters:
          project: ${{ parameters.testProjects }}
      - task: DotNetCoreCLI@2
        inputs:
          command: test
          projects: ${{ parameters.testProjects }}
```

### Variable Template
```yaml
# templates/variables/common.yml
parameters:
  - name: environment
    type: string
    values: [dev, staging, production]

variables:
  isProduction: ${{ eq(parameters.environment, 'production') }}
  artifactName: drop-${{ parameters.environment }}
  serviceConnection: sc-${{ parameters.environment }}
```

## Parameter Types and Validation

All supported parameter types: `string`, `boolean`, `number`, `object`, `step`, `stepList`, `job`, `jobList`, `stage`, `stageList`. Use `values:` to constrain string inputs at compile time.

```yaml
parameters:
  - name: environment
    type: string
    values: [dev, staging, production]   # Compile-time validation — invalid values fail parse
  - name: runTests
    type: boolean
    default: true
  - name: replicas
    type: number
    default: 3
  - name: extraSteps
    type: stepList
    default: []
  - name: poolConfig
    type: object
    default:
      vmImage: ubuntu-latest
```

## Conditional Insertion

Use `${{ if }}`, `${{ elseif }}`, `${{ else }}` to conditionally insert YAML blocks. These are compile-time — the YAML is included or excluded before the pipeline runs.

```yaml
stages:
  - stage: Deploy
    jobs:
      - deployment: DeployWeb
        environment: ${{ parameters.environment }}
        strategy:
          runOnce:
            deploy:
              steps:
                - ${{ if eq(parameters.environment, 'production') }}:
                  - task: AzureCLI@2
                    displayName: Pre-deploy health check
                    inputs:
                      azureSubscription: $(serviceConnection)
                      scriptType: bash
                      scriptLocation: inlineScript
                      inlineScript: |
                        az webapp show --name $(appName) --query state -o tsv
                - ${{ elseif eq(parameters.environment, 'staging') }}:
                  - script: echo "Deploying to staging — no health check required"
                - ${{ else }}:
                  - script: echo "Dev deployment"
```

## Iterative Insertion with `${{ each }}`

Loop over arrays or object key-value pairs to generate repeated YAML blocks.

```yaml
# Deploy to multiple regions dynamically
parameters:
  - name: regions
    type: object
    default:
      - name: eastus
        shortName: eus
      - name: westus2
        shortName: wus2

stages:
  - ${{ each region in parameters.regions }}:
    - stage: Deploy_${{ region.shortName }}
      displayName: Deploy to ${{ region.name }}
      jobs:
        - deployment: Deploy
          environment: production-${{ region.name }}
          strategy:
            runOnce:
              deploy:
                steps:
                  - script: echo "Deploying to ${{ region.name }}"
```

### Iterating Object Key-Value Pairs

```yaml
# Wrap user-provided jobs with pre/post steps (security injection pattern)
parameters:
  - name: jobs
    type: jobList
    default: []

jobs:
  - ${{ each job in parameters.jobs }}:
    - ${{ each pair in job }}:
        ${{ if ne(pair.key, 'steps') }}:
          ${{ pair.key }}: ${{ pair.value }}
      steps:
        - task: CredScan@1                 # Injected pre-step
        - ${{ job.steps }}                 # User's original steps
        - task: PublishMyTelemetry@1       # Injected post-step
          condition: always()
```

## Cross-Repository Template References

Reference templates from other repositories using `resources.repositories` and `template: file@repoAlias`.

```yaml
resources:
  repositories:
    - repository: templates
      type: git
      name: MyProject/pipeline-templates    # Azure Repos: project/repo
      ref: refs/tags/v2.1                   # Pin to tag for stability

stages:
  - template: stages/deploy.yml@templates
    parameters:
      environment: production
      serviceConnection: sc-prod
```

**Best practices for cross-repo templates:**
- Pin to a tag or specific commit (`ref: refs/tags/v2.1`), never float on a branch in production
- Use `endpoint:` for GitHub-hosted template repos with a service connection
- Template nesting works across repos: a template in repo A can reference templates in the same repo A, but NOT in another external repo — only the root pipeline can declare `resources.repositories`

## Production Stage Template Example

```yaml
# templates/stages/deploy-to-environment.yml
parameters:
  - name: environment
    type: string
    values: [dev, staging, production]
  - name: azureSubscription
    type: string
  - name: appName
    type: string
  - name: dependsOn
    type: object
    default: []
  - name: preDeploySteps
    type: stepList
    default: []

stages:
  - stage: Deploy_${{ parameters.environment }}
    displayName: Deploy to ${{ parameters.environment }}
    dependsOn: ${{ parameters.dependsOn }}
    condition: succeeded()
    variables:
      - template: ../variables/common.yml
        parameters:
          environment: ${{ parameters.environment }}
    jobs:
      - deployment: DeployApp
        displayName: Deploy ${{ parameters.appName }}
        environment: ${{ parameters.environment }}
        pool:
          vmImage: ubuntu-latest
        strategy:
          runOnce:
            deploy:
              steps:
                - ${{ parameters.preDeploySteps }}
                - task: AzureWebApp@1
                  displayName: Deploy to Azure Web App
                  inputs:
                    azureSubscription: ${{ parameters.azureSubscription }}
                    appName: ${{ parameters.appName }}
                    package: $(Pipeline.Workspace)/drop/*.zip
```

## Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad | Better Approach |
|---|---|---|
| Deeply nested templates (4+ levels) | Hard to debug, compile errors are cryptic | Flatten to max 2-3 levels |
| Floating `ref: refs/heads/main` on shared templates | Template changes break all consumers silently | Pin to tags or commit SHAs |
| Giant monolithic templates with 20+ parameters | Unmaintainable, defeats reuse purpose | Compose smaller single-responsibility templates |
| Using `${{ variables.foo }}` for runtime values | Variables aren't available at compile time unless set in YAML | Use `${{ parameters.foo }}` for compile-time, `$(foo)` for runtime |
| Passing secrets as template parameters | Secrets can leak in template expansion logs | Pass secret variable names, reference with `$(secretName)` at runtime |
