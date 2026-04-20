# Variable Group Migration

## ADO Variable Group Types

Azure DevOps has two types of variable groups:
1. **Plain variable groups** — key-value pairs stored in ADO Library
2. **Key Vault-linked variable groups** — variables fetched from Azure Key Vault at runtime

## Plain Variable Groups → GitHub Equivalents

### Non-Secret Variables → Repository or Organization Variables

ADO pipeline referencing a variable group:
```yaml
# ADO
variables:
  - group: app-config
# Contains: APP_NAME=myapp, REGION=eastus, NODE_VERSION=20

steps:
  - script: echo $(APP_NAME) in $(REGION)
```

GHA equivalent using repository variables:
```yaml
# GHA — set via Settings > Secrets and Variables > Variables
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "${{ vars.APP_NAME }} in ${{ vars.REGION }}"
```

**Scope mapping:**
| ADO Scope | GHA Equivalent |
|---|---|
| Variable group (project-wide) | Organization variables or repository variables |
| Pipeline-level variables | Workflow-level `env:` block |
| Stage-level variables | Job-level `env:` block |
| Job-level variables | Job-level `env:` block |

### Secret Variables → Repository or Organization Secrets

```yaml
# ADO — secret variable in group
variables:
  - group: credentials  # Contains secret: DB_PASSWORD

steps:
  - script: deploy.sh
    env:
      DB_PASSWORD: $(DB_PASSWORD)

# GHA — using repository secrets
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
        env:
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

### Environment-Specific Variables → GitHub Environment Variables/Secrets

ADO variable groups scoped to stages map to GHA environment variables:

```yaml
# GHA — environment-level variables and secrets
jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment: production  # Unlocks environment-scoped vars/secrets
    steps:
      - run: |
          echo "Deploying ${{ vars.APP_NAME }}"      # env variable
          echo "Using ${{ secrets.DB_PASSWORD }}"      # env secret
```

## Key Vault-Linked Variable Groups

ADO Key Vault-linked variable groups have **no direct GHA equivalent**. Three migration options:

### Option 1: azure/login + azure/CLI (Recommended with OIDC)

```yaml
permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Retrieve secrets from Key Vault
        id: kv
        uses: azure/CLI@v2
        with:
          inlineScript: |
            DB_PASS=$(az keyvault secret show --name db-password \
              --vault-name my-vault --query value -o tsv)
            echo "::add-mask::$DB_PASS"
            echo "DB_PASSWORD=$DB_PASS" >> $GITHUB_OUTPUT

      - name: Deploy
        run: ./deploy.sh
        env:
          DB_PASSWORD: ${{ steps.kv.outputs.DB_PASSWORD }}
```

### Option 2: Copy to GitHub Environment Secrets

For simpler setups, copy Key Vault values to GitHub environment secrets:
```bash
# One-time migration script
SECRETS=$(az keyvault secret list --vault-name my-vault --query "[].name" -o tsv)
for SECRET_NAME in $SECRETS; do
  VALUE=$(az keyvault secret show --name $SECRET_NAME --vault-name my-vault --query value -o tsv)
  gh secret set "${SECRET_NAME//-/_}" --body "$VALUE" --env production
done
```

**Trade-off:** Secrets are now duplicated — Key Vault updates won't auto-propagate to GitHub.

### Option 3: Inline az keyvault Commands

```yaml
- name: Get and use secret
  uses: azure/CLI@v2
  with:
    inlineScript: |
      export DB_PASS=$(az keyvault secret show --name db-password \
        --vault-name my-vault --query value -o tsv)
      echo "::add-mask::$DB_PASS"
      ./deploy.sh
```

## Variable Scoping Migration

ADO supports variable scoping at pipeline, stage, job levels:

```yaml
# ADO scoping
variables:
  globalVar: pipeline-level       # Available everywhere

stages:
  - stage: Build
    variables:
      stageVar: build-stage       # Available in this stage only
    jobs:
      - job: Compile
        variables:
          jobVar: compile-job     # Available in this job only

# GHA equivalent
env:
  GLOBAL_VAR: pipeline-level      # Workflow-level

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      STAGE_VAR: build-stage      # Job-level
    steps:
      - run: echo "all visible"
        env:
          JOB_VAR: compile-job    # Step-level
```

## Queue-Time Variables → workflow_dispatch Inputs

ADO allows users to set variable values at queue time. GHA equivalent is `workflow_dispatch`:

```yaml
# ADO
parameters:
  - name: deployEnvironment
    type: string
    default: dev
    values: [dev, staging, prod]
  - name: skipTests
    type: boolean
    default: false

# GHA
on:
  workflow_dispatch:
    inputs:
      deploy-environment:
        description: Target environment
        required: true
        default: dev
        type: choice
        options: [dev, staging, prod]
      skip-tests:
        description: Skip test execution
        type: boolean
        default: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to ${{ inputs.deploy-environment }}"
```

**Limitations vs ADO runtime parameters:**
- No `object` type — must use `string` with JSON parsing
- No stage-level parameters — inputs are workflow-global only
- No `values` constraint on string type — use `choice` type instead
- Cannot set at the job or stage level — only top-level workflow inputs

## settableVariables → No GHA Equivalent

ADO `settableVariables` restricts which variables a step can set:

```yaml
# ADO
steps:
  - bash: echo "##vso[task.setvariable variable=myVar]value"
    target:
      settableVariables:
        - myVar  # Only myVar can be set
```

GHA has no equivalent restriction. Any step can write to `$GITHUB_ENV` or `$GITHUB_OUTPUT`.
Mitigate by using environment-level access controls and required reviewers on environments.

## Library Audit via REST API

Before migrating, inventory all variable groups in the ADO project:

```bash
# List all variable groups
curl -u :$ADO_PAT \
  "https://dev.azure.com/{org}/{project}/_apis/distributedtask/variablegroups?api-version=7.1" \
  | jq '.value[] | {
    name,
    type,
    variableCount: (.variables | length),
    hasSecrets: ([.variables[].isSecret // false] | any),
    keyVaultLinked: (.providerData != null)
  }'
```

## Migration Checklist

1. **Inventory** — List all variable groups and their variables via REST API
2. **Classify** — Categorize each variable: secret vs non-secret, static vs environment-specific
3. **Map Key Vault groups** — Decide: fetch at runtime (Option 1) or copy to GitHub (Option 2)
4. **Create GitHub targets** — Set up org variables, repo variables, environment variables/secrets
5. **Update workflows** — Replace `$(VAR_NAME)` with `${{ vars.VAR_NAME }}` or `${{ secrets.VAR_NAME }}`
6. **Validate** — Run workflows and verify all variables resolve correctly
7. **Secret redaction** — Verify secret masking in GHA logs (GHA masks `secrets.*` automatically,
   but runtime-fetched Key Vault secrets need explicit `::add-mask::`)

## Common Mistakes

- **Forgetting `::add-mask::`** for dynamically fetched secrets — they will appear in logs
- **Key Vault access policies** — new OIDC service principal needs `Key Vault Secrets User` role
- **Variable name casing** — ADO variables are case-insensitive; GHA `vars.*` and `secrets.*` are case-sensitive
- **Secret in `env:` vs `with:`** — Secrets passed via `env:` are masked; secrets interpolated
  directly in `run:` commands may appear in process listings
- **Group authorization** — ADO variable groups can be authorized per-pipeline; GHA variables/secrets
  have different access models (org-level requires visibility settings, env-level requires environment reference)
