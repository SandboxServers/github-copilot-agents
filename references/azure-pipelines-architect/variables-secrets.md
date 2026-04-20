# Azure Pipelines Variables and Secrets

## Variable Definition Levels

Variables can be defined at four levels in YAML, with specific scoping rules.

```yaml
# Pipeline-level variables
variables:
  globalVar: 'available-everywhere'

stages:
  - stage: Build
    variables:
      stageVar: 'available-in-this-stage'   # Stage-scoped
    jobs:
      - job: Compile
        variables:
          jobVar: 'available-in-this-job'    # Job-scoped
        steps:
          - script: echo $(globalVar) $(stageVar) $(jobVar)
          - bash: |
              echo "##vso[task.setvariable variable=stepVar]only-this-job"  # Step-set: job-scoped
```

## Variable Precedence (Highest to Lowest)

When the same variable name is set at multiple levels, the **most specific** scope wins:

1. **Queue-time variables** (set when manually running) — highest precedence
2. **Step-level** `task.setvariable` — within the current job
3. **Job-level** variables
4. **Stage-level** variables
5. **Pipeline-level** variables (YAML root)
6. **Variable groups** — linked at pipeline level
7. **Pipeline settings UI** variables
8. **Predefined variables** — lowest precedence

**Important:** Queue-time variables override YAML-defined variables. Use `settableVariables` to restrict which variables can be set at queue time:

```yaml
jobs:
  - job: Build
    variables:
      buildConfig: Release
    pool:
      vmImage: ubuntu-latest
    settableVariables:
      - buildConfig                         # Only this variable can be overridden at queue time
    steps:
      - script: echo $(buildConfig)
```

## Variable Groups

Variable groups store sets of variables that can be shared across pipelines.

```yaml
variables:
  - group: common-settings                  # Link a variable group
  - group: secrets-keyvault                 # Variable group linked to Key Vault
  - name: localVar                          # Inline variable alongside groups
    value: 'hello'
```

### Key Vault-Linked Variable Groups

Link a variable group directly to an Azure Key Vault to pull secrets at pipeline runtime.

**Setup requirements:**
1. Create a service connection to Azure (Azure Resource Manager)
2. The service connection's service principal needs **Get** and **List** permissions on Key Vault secrets
3. In Azure DevOps Library, create a variable group linked to the Key Vault
4. Authorize specific secrets to be pulled — not all secrets are exposed by default

```yaml
variables:
  - group: kv-linked-group                  # Contains secrets from Key Vault

steps:
  - task: AzureCLI@2
    inputs:
      azureSubscription: my-azure-sc
      scriptType: bash
      inlineScript: |
        # Key Vault secrets are available as variables
        echo "Using connection string..."    # Don't echo actual secrets!
    env:
      DB_CONNECTION: $(dbConnectionString)   # Map secret to env var
      API_KEY: $(apiKey)                     # Secrets must be explicitly mapped to env
```

## Secret Variables

### Behavior of Secrets
- **Redacted in logs** — the agent masks secret values in all log output
- **Not passed to forked PR builds** — secrets are unavailable in PR builds from forks
- **Not decrypted automatically** — must be explicitly mapped via `env:` or `$(varName)`
- **Cannot be used in template expressions** — `${{ variables.secret }}` won't work for secrets

### Setting Secrets in Scripts
```yaml
steps:
  - bash: |
      echo "##vso[task.setvariable variable=mySecret;isSecret=true]$(sensitiveValue)"
    displayName: Set secret from computation

  - bash: |
      # Access the secret — must map via env for bash
      echo "The secret length is ${#MY_SECRET}"
    env:
      MY_SECRET: $(mySecret)
```

## Output Variables (Cross-Job and Cross-Stage)

### Within a Stage (Cross-Job)
```yaml
jobs:
  - job: Producer
    steps:
      - bash: echo "##vso[task.setvariable variable=version;isOutput=true]2.1.$(Build.BuildId)"
        name: setVer                        # 'name' is required for output variables

  - job: Consumer
    dependsOn: Producer
    variables:
      producedVersion: $[dependencies.Producer.outputs['setVer.version']]
    steps:
      - script: echo "Version is $(producedVersion)"
```

### Cross-Stage
```yaml
stages:
  - stage: Build
    jobs:
      - job: BuildJob
        steps:
          - bash: echo "##vso[task.setvariable variable=imageTag;isOutput=true]$(Build.BuildId)"
            name: setTag

  - stage: Deploy
    dependsOn: Build
    variables:
      tag: $[stageDependencies.Build.BuildJob.outputs['setTag.imageTag']]
    jobs:
      - job: DeployJob
        steps:
          - script: echo "Deploying tag $(tag)"
```

**Key rules for output variables:**
- The step setting the variable **must** have a `name:` property
- Cross-job: `$[dependencies.<JobName>.outputs['<StepName>.<VarName>']]`
- Cross-stage: `$[stageDependencies.<StageName>.<JobName>.outputs['<StepName>.<VarName>']]`
- In deployment jobs, add the resource name: `$[dependencies.<JobName>.outputs['<ResourceName>.<StepName>.<VarName>']]`

## Variable Templates

Define reusable variable sets in templates, optionally parameterized.

```yaml
# templates/variables/environment-vars.yml
parameters:
  - name: env
    type: string
    values: [dev, staging, production]

variables:
  environment: ${{ parameters.env }}
  resourceGroup: rg-myapp-${{ parameters.env }}
  appServicePlan: asp-myapp-${{ parameters.env }}
  ${{ if eq(parameters.env, 'production') }}:
    sku: P1v3
    replicas: 3
  ${{ else }}:
    sku: B1
    replicas: 1
```

```yaml
# azure-pipelines.yml — consuming the variable template
variables:
  - template: templates/variables/environment-vars.yml
    parameters:
      env: production
```

## Runtime vs Compile-Time Variable Expansion

| Syntax | When Expanded | Use Case |
|---|---|---|
| `${{ variables.varName }}` | Compile time — YAML parsing | Only works for variables defined inline in YAML |
| `$(varName)` | Task execution — agent macro expansion | Task inputs, scripts — most common usage |
| `$[variables['varName']]` | Runtime — job/stage execution | Conditions, variable definitions using dependencies |

## Predefined Variables — Most Useful

| Variable | Description | Example Value |
|---|---|---|
| `Build.BuildId` | Unique numeric build ID | `1247` |
| `Build.BuildNumber` | Human-readable build number | `20240115.3` |
| `Build.SourceBranch` | Full ref of triggering branch | `refs/heads/main` |
| `Build.SourceBranchName` | Short branch name | `main` |
| `Build.Repository.Name` | Repository name | `MyApp` |
| `Build.Reason` | Why the build ran | `IndividualCI`, `PullRequest`, `Manual`, `Schedule` |
| `System.PullRequest.PullRequestId` | PR number (if PR-triggered) | `42` |
| `System.PullRequest.SourceBranch` | PR source branch | `refs/heads/feature/x` |
| `System.PullRequest.TargetBranch` | PR target branch | `refs/heads/main` |
| `Agent.OS` | Agent operating system | `Linux`, `Windows_NT`, `Darwin` |
| `Pipeline.Workspace` | Root workspace directory | `/home/vsts/work/1` |
| `Build.ArtifactStagingDirectory` | Staging dir for publish | `/home/vsts/work/1/a` |

## Common Mistakes

| Mistake | Problem | Fix |
|---|---|---|
| `${{ variables.queueTimeVar }}` | Queue-time vars aren't available at compile time | Use `$(queueTimeVar)` or `$[variables['queueTimeVar']]` |
| Secret in compile-time expression | Secrets can't be used in `${{ }}` | Use `$(secretName)` or map via `env:` |
| Output variable without `name:` on step | Variable is set but not accessible | Add `name: myStep` to the step |
| Output variable in deployment job | Different dependency path for deployment jobs | Include resource name in the expression path |
| Variable group not authorized | Pipeline can't read group variables | Authorize the pipeline in Library settings |
| `isOutput=true` but wrong dependency syntax | Cross-job vs cross-stage syntax differs | Use `dependencies.*` for jobs, `stageDependencies.*` for stages |
