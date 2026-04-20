# What-If & Linter

## What-If Deployments
Preview changes before deploying — see what will be created, modified, or deleted.

```bash
# Resource group scope
az deployment group what-if \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters parameters/prod.bicepparam

# Subscription scope
az deployment sub what-if \
  --location eastus2 \
  --template-file platform.bicep \
  --parameters parameters/platform.bicepparam

# Management group scope
az deployment mg what-if \
  --management-group-id myMgGroup \
  --location eastus2 \
  --template-file policies.bicep

# Tenant scope
az deployment tenant what-if \
  --location eastus2 \
  --template-file hierarchy.bicep
```

### Change Types
| Symbol | Change Type | Meaning |
|---|---|---|
| `+` (green) | Create | Resource will be created |
| `~` (purple) | Modify | Resource exists, properties will change |
| `-` (red) | Delete | Resource will be deleted (complete mode only) |
| `*` (gray) | NoChange | Resource exists, no changes detected |
| `x` (yellow) | Ignore | Resource is outside the deployment scope |
| `!` (orange) | NoEffect | No change due to deployment condition |

### Filtering What-If Output
```bash
# Show only creates and deletes
az deployment group what-if \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters parameters/prod.bicepparam \
  --result-format FullResourcePayloads    # See complete resource definitions

# JSON output for CI/CD parsing
az deployment group what-if \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters parameters/prod.bicepparam \
  --no-pretty-print -o json
```

### CI/CD What-If Pattern
```yaml
# Azure Pipelines — what-if as PR gate
- task: AzureCLI@2
  displayName: 'What-If Analysis'
  inputs:
    azureSubscription: 'production-wif'
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      result=$(az deployment group what-if \
        --resource-group $(resourceGroup) \
        --template-file main.bicep \
        --parameters parameters/$(environment).bicepparam \
        --no-pretty-print -o json)
      
      # Check for destructive changes
      deletes=$(echo "$result" | jq '[.changes[] | select(.changeType == "Delete")] | length')
      if [ "$deletes" -gt "0" ]; then
        echo "##vso[task.logissue type=warning]What-If shows $deletes resources will be DELETED"
      fi
      
      echo "$result" | jq '.changes[] | {resourceId: .resourceId, changeType: .changeType}'
```

## Bicep Linter
The Bicep linter runs automatically on build and in the VS Code extension. Configure it in `bicepconfig.json`.

### bicepconfig.json
```json
{
  "analyzers": {
    "core": {
      "enabled": true,
      "rules": {
        "no-hardcoded-env-urls": {
          "level": "error"
        },
        "no-unused-params": {
          "level": "error"
        },
        "no-unused-vars": {
          "level": "error"
        },
        "prefer-interpolation": {
          "level": "warning"
        },
        "secure-parameter-default": {
          "level": "error"
        },
        "simplify-interpolation": {
          "level": "warning"
        },
        "protect-commandtoexecute-secrets": {
          "level": "error"
        },
        "use-stable-vm-image": {
          "level": "warning"
        },
        "use-safe-access": {
          "level": "warning"
        },
        "no-hardcoded-location": {
          "level": "error"
        },
        "no-unnecessary-dependson": {
          "level": "warning"
        },
        "use-recent-api-versions": {
          "level": "warning",
          "maxAllowedAgeInDays": 730
        },
        "use-resource-id-functions": {
          "level": "warning"
        },
        "explicit-values-for-loc-params": {
          "level": "warning"
        },
        "max-outputs": {
          "level": "warning"
        },
        "max-params": {
          "level": "warning"
        },
        "max-resources": {
          "level": "warning"
        },
        "max-variables": {
          "level": "warning"
        },
        "outputs-should-not-contain-secrets": {
          "level": "error"
        }
      }
    }
  }
}
```

### Key Linter Rules Explained

| Rule | Level | What It Catches |
|---|---|---|
| `no-hardcoded-env-urls` | error | URLs like `management.azure.com` instead of `environment().resourceManager` |
| `no-unused-params` | error | Parameters declared but never referenced |
| `no-unused-vars` | error | Variables declared but never referenced |
| `secure-parameter-default` | error | Secure parameters with default values (the default shows in deployment history) |
| `no-hardcoded-location` | error | Location strings like `'eastus2'` instead of `param location` |
| `outputs-should-not-contain-secrets` | error | Outputs that expose Key Vault references or secure values |
| `use-recent-api-versions` | warning | API versions older than 2 years |
| `no-unnecessary-dependson` | warning | Explicit `dependsOn` that Bicep already infers from symbolic references |
| `prefer-interpolation` | warning | `concat()` instead of string interpolation `'${}'` |

### Suppressing Linter Warnings
```bicep
// Suppress for a single line
#disable-next-line no-hardcoded-location
param defaultLocation string = 'eastus2'    // Justified: this is a module default

// Suppress with explanation (preferred)
#disable-next-line no-hardcoded-env-urls // Using sovereign cloud URL for government deployment
var managementUrl = 'https://management.usgovcloudapi.net'
```

## Bicep Build & Validate
```bash
# Build — transpile to ARM JSON (catches syntax errors)
az bicep build --file main.bicep

# Build with stdout (for piping)
az bicep build --file main.bicep --stdout

# Validate against Azure — checks resource provider requirements
az deployment group validate \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters parameters/prod.bicepparam

# Preflight validation (validates without creating deployment object)
az deployment group validate \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters parameters/prod.bicepparam

# Decompile ARM JSON to Bicep (migration tool)
az bicep decompile --file template.json
```

## Validation Pipeline Order
Run these in order — each catches different issues:

1. **`az bicep build`** — Syntax errors, type mismatches, linter violations
2. **`az deployment group validate`** — ARM validation, resource provider checks
3. **`az deployment group what-if`** — Preview actual changes against live environment
4. **`az deployment group create`** — Deploy (only after all checks pass)
