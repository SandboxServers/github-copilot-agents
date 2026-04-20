# Template Migration Patterns

## ADO Stage Templates → GHA Reusable Workflows

ADO stage templates encapsulate entire stages with jobs. The GHA equivalent is a **reusable workflow**
called with `uses:` at the job level.

### ADO Stage Template

```yaml
# templates/deploy-stage.yml
parameters:
  - name: environment
    type: string
  - name: azureSubscription
    type: string
  - name: appName
    type: string

stages:
  - stage: Deploy_${{ parameters.environment }}
    condition: succeeded()
    jobs:
      - deployment: deploy
        environment: ${{ parameters.environment }}
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: ${{ parameters.azureSubscription }}
                    appName: ${{ parameters.appName }}
```

### GHA Reusable Workflow Equivalent

```yaml
# .github/workflows/deploy-reusable.yml
name: Deploy
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      app-name:
        required: true
        type: string
    secrets:
      AZURE_CLIENT_ID:
        required: true
      AZURE_TENANT_ID:
        required: true
      AZURE_SUBSCRIPTION_ID:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - uses: azure/webapps-deploy@v3
        with:
          app-name: ${{ inputs.app-name }}
```

**Caller workflow:**
```yaml
jobs:
  deploy-staging:
    uses: my-org/shared-workflows/.github/workflows/deploy-reusable.yml@main
    with:
      environment: staging
      app-name: myapp-staging
    secrets: inherit  # Or pass explicitly
```

## ADO Job Templates → GHA Reusable Workflows

Job templates map similarly — one ADO job template becomes one reusable workflow.

Key differences:
- ADO job templates can define `container:` and `services:` — GHA reusable workflows support
  these at the job level via `container:` and `services:` in the reusable workflow itself
- ADO `pool:` → GHA `runs-on:` (pass as input if variable)

## ADO Step Templates → GHA Composite Actions

Step templates that define a reusable set of steps become **composite actions**.

### ADO Step Template

```yaml
# templates/npm-build-steps.yml
parameters:
  - name: nodeVersion
    type: string
    default: '20'
  - name: buildCommand
    type: string
    default: 'build'

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: ${{ parameters.nodeVersion }}
  - script: npm ci
    displayName: Install dependencies
  - script: npm run ${{ parameters.buildCommand }}
    displayName: Build
```

### GHA Composite Action Equivalent

```yaml
# .github/actions/npm-build/action.yml
name: NPM Build
description: Install and build Node.js project
inputs:
  node-version:
    description: Node.js version
    default: '20'
  build-command:
    description: npm script to run
    default: 'build'

runs:
  using: composite
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
    - run: npm ci
      shell: bash
    - run: npm run ${{ inputs.build-command }}
      shell: bash
```

**Key difference:** Composite actions require `shell:` on every `run:` step.

## ADO Variable Templates → GHA Equivalents

ADO variable templates centralize variable definitions. GHA has several alternatives:

```yaml
# ADO variable template
# templates/variables.yml
variables:
  appName: myapp
  region: eastus
  nodeVersion: '20'
```

**GHA alternatives:**
1. **Workflow-level env block** — for workflow-scoped values
2. **Organization/repository variables** (`vars.APP_NAME`) — for shared non-secrets
3. **Environment variables** — for environment-specific values
4. **Reusable workflow inputs** — for parameterized values

```yaml
# GHA: workflow-level env
env:
  APP_NAME: myapp
  REGION: eastus
  NODE_VERSION: '20'
```

## Cross-Repo Template References

### ADO Cross-Repo Templates

```yaml
# ADO: reference template from another repo
resources:
  repositories:
    - repository: templates
      type: git
      name: SharedProject/pipeline-templates
      ref: refs/heads/main

stages:
  - template: stages/deploy.yml@templates
    parameters:
      environment: production
```

### GHA Cross-Repo Reusable Workflows

```yaml
# GHA: reference reusable workflow from another repo
jobs:
  deploy:
    uses: my-org/shared-workflows/.github/workflows/deploy.yml@main
    with:
      environment: production
    secrets: inherit
```

**Key differences:**
- ADO uses `template@repo` syntax → GHA uses `org/repo/.github/workflows/file.yml@ref`
- ADO templates can be from any Azure Repos Git repo → GHA reusable workflows must be in
  a public repo, or the same org (for internal/private repos with org-level setting enabled)
- ADO ref can be branch or tag → GHA ref can be branch, tag, or full SHA (SHA recommended)

## Template Parameter Type Mapping

| ADO Parameter Type | GHA Equivalent | Notes |
|---|---|---|
| `string` | `string` (input) | Direct mapping |
| `boolean` | `boolean` (input) | Direct mapping |
| `number` | `number` (input) | Direct mapping for reusable workflows |
| `object` | `string` + `fromJSON()` | Pass JSON string, parse with `fromJSON()` |
| `stepList` | Composite action | No direct param type — use separate composite action |
| `jobList` | Separate reusable workflows | No direct param type — restructure as multiple calls |

### Object Parameter Pattern

```yaml
# ADO: object parameter
parameters:
  - name: deployTargets
    type: object
    default:
      - name: eastus
        slots: [staging, production]

# GHA: pass as JSON string input
on:
  workflow_call:
    inputs:
      deploy-targets:
        type: string  # JSON string

jobs:
  deploy:
    strategy:
      matrix:
        target: ${{ fromJSON(inputs.deploy-targets) }}
    steps:
      - run: echo "Deploying to ${{ matrix.target.name }}"
```

## Conditional Insertion Migration

### ADO ${{ if }} → GHA if:

```yaml
# ADO conditional insertion
steps:
  - ${{ if eq(parameters.runTests, true) }}:
    - script: npm test
      displayName: Run tests

# GHA conditional step
steps:
  - name: Run tests
    if: inputs.run-tests == true
    run: npm test
```

**Critical syntax difference:** ADO `${{ if }}` is compile-time (template expansion).
GHA `if:` is runtime evaluation. ADO `${{ if }}` can conditionally insert entire jobs
or steps — GHA `if:` only skips/runs existing steps.

## Iterative Insertion (${{ each }})

**ADO `each` has NO direct GHA equivalent.** Migration strategies:

### Strategy 1: Matrix (for parallel execution)

```yaml
# ADO each
parameters:
  - name: regions
    type: object
    default: [eastus, westus, centralus]
steps:
  - ${{ each region in parameters.regions }}:
    - script: deploy.sh ${{ region }}

# GHA matrix
jobs:
  deploy:
    strategy:
      matrix:
        region: [eastus, westus, centralus]
    steps:
      - run: deploy.sh ${{ matrix.region }}
```

### Strategy 2: Duplicate steps (for sequential execution)

When `each` is used for sequential steps that must run in order, you must
explicitly list each step — there is no looping construct in GHA.

## Template Nesting Limits

- **ADO**: Templates can nest deeply (practically unlimited)
- **GHA**: Reusable workflows can nest up to **4 levels deep**
  (a reusable workflow can call another reusable workflow, up to 4 hops)
- **GHA**: Maximum of **20 reusable workflows** per workflow file

If ADO templates are deeply nested, flatten the hierarchy during migration.

## Common Pitfalls

1. **`strategy` + `uses` exclusion**: A job using `strategy: matrix` CANNOT also use
   `uses:` (reusable workflow). Matrix must be defined in the calling workflow, not the reusable one.

2. **Secret propagation**: Reusable workflows don't automatically receive caller secrets.
   Use `secrets: inherit` or pass each secret explicitly.

3. **Output passing complexity**: Reusable workflow outputs must be explicitly declared
   and mapped through multiple levels — much more verbose than ADO template outputs.

4. **Composite action limitations**: Composite actions cannot use `services:`, `container:`,
   or `if:` conditions on the composite action itself (only on individual steps within).

5. **No conditional job insertion**: GHA cannot conditionally include/exclude entire jobs
   via template logic — use `if:` conditions on jobs instead.

6. **Shell requirement**: Every `run:` step in a composite action MUST specify `shell:`.
   This is the most common error when converting ADO step templates.
