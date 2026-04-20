# Bicep Anti-Patterns

Patterns that compile but lead to fragile, insecure, or unmaintainable deployments.

## Hardcoding Values That Should Be Parameters
```bicep
// BAD — locked to one region, no reuse
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: 'stprodeastus2app01'
  location: 'eastus2'
  sku: { name: 'Standard_GRS' }
  kind: 'StorageV2'
}

// GOOD — parameterized
param location string = resourceGroup().location
param storageAccountName string
param skuName string = 'Standard_LRS'
```

## Deploying Resources That Aren't Needed
```bicep
// BAD — always deploys diagnostics even when not wanted
resource diagStorage 'Microsoft.Storage/storageAccounts@2023-05-01' = { ... }

// GOOD — conditional deployment
param deployDiagnostics bool = false
resource diagStorage 'Microsoft.Storage/storageAccounts@2023-05-01' = if (deployDiagnostics) { ... }
```

## Resource Name Collisions
```bicep
// BAD — will collide across environments and subscriptions
var storageName = 'stmyapp'

// GOOD — globally unique via uniqueString()
var storageName = 'st${take(project, 6)}${uniqueString(resourceGroup().id)}'
```

## Ignoring Deployment Scope Limitations
```bicep
// BAD — resource group-scoped template trying to create a resource group
targetScope = 'resourceGroup'
resource rg 'Microsoft.Resources/resourceGroups@2024-03-01' = { ... } // ERROR

// GOOD — use subscription scope for resource group creation
targetScope = 'subscription'
resource rg 'Microsoft.Resources/resourceGroups@2024-03-01' = { ... }
```

## Overly Complex Nested Modules
```bicep
// BAD — three layers of nesting for a single storage account
module infra './modules/infra/main.bicep'          // calls →
  module storage './modules/infra/storage/main.bicep'  // calls →
    module account './modules/infra/storage/account.bicep'

// GOOD — flat is clearer when nesting adds no reuse value
module storage './modules/storage-account.bicep' = { ... }
```

## Not Using `existing` for Pre-Existing Resources
```bicep
// BAD — re-deploying a resource you don't own just to get its ID
resource vnet 'Microsoft.Network/virtualNetworks@2024-05-01' = {
  name: 'vnet-shared'
  location: location
  properties: { ... } // Risk: overwrites config you don't control
}

// GOOD — reference without modifying
resource vnet 'Microsoft.Network/virtualNetworks@2024-05-01' existing = {
  name: 'vnet-shared'
  scope: resourceGroup(networkRgName)
}
```

## Ignoring Idempotency
```bicep
// BAD — deployment name with timestamp fails on re-run if ARM caches it
module app './modules/app.bicep' = {
  name: 'deploy-${utcNow()}'   // Different name each run = orphaned deployments
}

// GOOD — deterministic deployment names
module app './modules/app.bicep' = {
  name: 'deploy-app-${environment}'
}
```

## Using Outputs for Secrets
```bicep
// BAD — secrets appear in deployment history (visible in Portal and API)
output sqlPassword string = sqlAdminPassword
output connectionString string = 'Server=${sql.properties.fullyQualifiedDomainName};Password=${password}'

// GOOD — store secrets in Key Vault, output the Key Vault secret URI
output sqlSecretUri string = keyVaultSecret.properties.secretUri
```

## Deploying Without What-If First
Always run `what-if` before applying changes, especially in production:
```bash
# Preview changes before deployment
az deployment group what-if --resource-group rg-prod --template-file main.bicep --parameters main.prod.bicepparam
```
Skipping what-if risks unintended deletes in `Complete` mode or unexpected property changes.

## Monolithic Template
```bicep
// BAD — 2000-line main.bicep with every resource inline
// Hard to test, review, reuse, or understand

// GOOD — modular structure
// main.bicep orchestrates modules
module networking './modules/networking.bicep' = { ... }
module database './modules/database.bicep' = { ... }
module appService './modules/app-service.bicep' = { ... }
```

## Not Validating Inputs
```bicep
// BAD — accepts any string, fails at deploy time with cryptic ARM error
param environment string

// GOOD — fail fast with clear validation
@allowed(['dev', 'staging', 'prod'])
param environment string

@minLength(3)
@maxLength(24)
param storageAccountName string

@minValue(1)
@maxValue(10)
param instanceCount int = 1
```

## Ignoring Linter Warnings
The Bicep linter catches real problems. Don't suppress without justification:
- `no-unused-params` — dead parameters confuse consumers
- `no-hardcoded-env-url` — breaks sovereign clouds
- `prefer-interpolation` — concat() is less readable than `'${var}'`
- `secure-parameter-default` — secure params must not have defaults
- `use-resource-symbol-reference` — use symbolic names, not resourceId()

Configure in `bicepconfig.json`, never blanket-disable:
```json
{
  "analyzers": {
    "core": {
      "rules": {
        "no-hardcoded-env-url": { "level": "error" }
      }
    }
  }
}
```
