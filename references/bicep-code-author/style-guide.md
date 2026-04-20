# Style Guide

Write Bicep that a tired human can read at 2am during an incident. No clever tricks. No showing off. Every file should be so obvious that the person reading it six months from now — who might be you — can understand it without a Rosetta Stone.

## Naming Conventions

Be consistent. Be predictable. Be boring. Boring is good.

- **Resources:** camelCase for symbolic names. Name them what they are. `storageAccount`, `keyVault`, `virtualNetwork`. Not `sa`, `kv`, or `vnet1`. You're not paying per character.
- **Parameters:** camelCase, descriptive enough to stand alone. `storageAccountName`, `location`, `environment`. If someone has to read the rest of the file to understand what a parameter does, the name is wrong.
- **Variables:** camelCase. `namePrefix`, `commonTags`, `subnetAddressPrefix`. Same rules — clarity over brevity.
- **Outputs:** camelCase, describe exactly what's coming out. `storageAccountId`, `principalId`, `connectionString`. The caller shouldn't have to guess.
- **Modules:** camelCase symbolic name in the parent file. File names use kebab-case. Example: `module storageAccount './modules/storage-account.bicep'`. The symbolic name is how you reference it in code; the file name is how you find it on disk.
- **User-defined types:** camelCase with a `Type` suffix. `subnetConfigType`, `tagConfigType`, `diagnosticSettingType`. The suffix tells everyone this is a shape definition, not an instance.

## File Layout (Standard Order)

Every Bicep file follows this order. Every time. No exceptions. If a section doesn't apply, skip it — but never reorder what's there.

1. `targetScope` — only if it's not `resourceGroup` (which is the default)
2. `metadata` — file description, always present
3. `import` statements
4. User-defined types (`type`)
5. User-defined functions (`func`)
6. Parameters (`param`) — required parameters first, then optional ones with defaults
7. Variables (`var`)
8. Resources and modules — ordered by dependency (things that are depended on come first)
9. Outputs (`output`)

### Complete Example

```bicep
targetScope = 'subscription'

metadata description = 'Deploys a resource group and a storage account for application data. Used by the data pipeline team.'

// ============================================================================
// Types
// ============================================================================

@description('Configuration shape for diagnostic settings.')
type diagnosticSettingType = {
  @description('Name of the diagnostic setting.')
  name: string

  @description('Log Analytics workspace resource ID to send logs to.')
  workspaceId: string
}

// ============================================================================
// Parameters
// ============================================================================

// --- Required ---

@description('Short name of the project. Used in resource naming.')
@minLength(2)
@maxLength(10)
param project string

@description('Name of the target environment.')
@allowed(['dev', 'staging', 'prod'])
param environment string

// --- Optional ---

@description('Azure region for all resources. Defaults to deployment location.')
param location string = deployment().location

@description('Optional diagnostic settings. If provided, logs will be forwarded.')
param diagnosticSettings diagnosticSettingType?

// ============================================================================
// Variables
// ============================================================================

var namePrefix = '${project}-${environment}'

var commonTags = {
  Environment: environment
  Project: project
  ManagedBy: 'Bicep'
  LastDeployed: utcNow('yyyy-MM-dd')
}

// ============================================================================
// Resources / Modules
// ============================================================================

// Deploy the resource group that will hold all project resources.
resource resourceGroup 'Microsoft.Resources/resourceGroups@2024-03-01' = {
  name: '${namePrefix}-rg'
  location: location
  tags: commonTags
}

// Deploy the storage account for application data inside the resource group.
module storageAccount './modules/storage-account.bicep' = {
  name: '${namePrefix}-storage-deploy'
  scope: resourceGroup
  params: {
    namePrefix: namePrefix
    location: location
    tags: commonTags
  }
}

// ============================================================================
// Outputs
// ============================================================================

@description('Resource ID of the deployed resource group.')
output resourceGroupId string = resourceGroup.id

@description('Resource ID of the deployed storage account.')
output storageAccountId string = storageAccount.outputs.storageAccountId
```

## Commenting Rules

Comments are not optional. They're a kindness to the next person. That next person is often you.

- **Every parameter** gets a `@description()` decorator. No exceptions. Not even for `location`. Especially not for `location`, because sometimes it doesn't mean what you think it means.
- **Every output** gets a `@description()` decorator. No exceptions. The caller needs to know what they're getting.
- **Every module call** gets a comment above it explaining what it deploys and why it exists. One or two lines is fine.
- **Complex expressions** get inline comments explaining the logic. If you had to think about it for more than five seconds, comment it.
- **Every file** starts with `metadata description = '...'` explaining what this file is for and who uses it.
- **Explain WHY, not WHAT.** `// Enable soft delete because compliance requires 90-day recovery` is useful. `// Set enableSoftDelete to true` is not.

## Decorator Usage

Decorators are your input validation layer. Use them aggressively. Catching a bad value at deploy time is infinitely better than catching it in a failed resource provisioning twenty minutes later.

```bicep
// Description on everything. No exceptions.
@description('Name of the storage account. Must be globally unique.')
@minLength(3)
@maxLength(24)
param storageAccountName string

// Constrain choices to prevent typos and bad values.
@description('SKU tier for the storage account.')
@allowed(['Standard_LRS', 'Standard_GRS', 'Standard_ZRS'])
param skuName string = 'Standard_LRS'

// Integers get range constraints when there are known limits.
@description('Number of days to retain deleted blobs.')
@minValue(1)
@maxValue(365)
param retentionDays int = 30

// Secrets are marked secure. Always. No discussion.
@description('Connection string for the backend database. Will not appear in logs or outputs.')
@secure()
param databaseConnectionString string

// Additional context when description alone isn't enough.
@description('Deployment region override.')
@metadata({ example: 'eastus2', note: 'Must match the region of the existing VNet.' })
param targetRegion string
```

## Parameter Design

- **Always** provide a default for `location`. Use `resourceGroup().location` for resource group scoped deployments, `deployment().location` for subscription scoped. Don't make callers pass a location unless they're doing something unusual.
- **Use `@allowed()`** for environment names, SKU tiers, and any parameter with a known set of valid values. Typos in parameter files cause silent misconfigurations.
- **Use `@secure()`** for passwords, connection strings, keys, and anything that should never appear in logs. **Never** output a secure value. **Never** put a secure value in a variable that gets logged. If you think you need to, you don't — redesign.
- **Group related parameters** with comment blocks. All networking params together, all naming params together.
- **Validate early.** Every constraint you can express as a decorator is a deployment failure you prevent before it wastes anyone's time.

## Resource Configuration Style

Keep resources visually consistent so your eyes can scan a file and know where things are.

- **One blank line** between resources. No more, no less.
- **Properties in a consistent order** within each resource: `name`, `location`, `tags`, `sku`, `kind`, `properties`, `identity`, `dependsOn`. Not every resource has all of these, but when they're present, they follow this order.
- **Always include tags** on anything that supports them. Use a `commonTags` variable and union with resource-specific tags if needed.
- **Use the `existing` keyword** to reference resources that already exist. Don't pass resource IDs around when you can reference the resource directly and let Bicep resolve the dependency.
- **Prefer symbolic references** over `dependsOn`. If resource B references a property of resource A, Bicep knows the dependency. Only use explicit `dependsOn` when there's an ordering requirement that isn't expressed through property references — and leave a comment explaining why.

```bicep
// Reference an existing Log Analytics workspace instead of passing its ID around.
resource logAnalytics 'Microsoft.OperationalInsights/workspaces@2023-09-01' existing = {
  name: logAnalyticsWorkspaceName
  scope: resourceGroup(monitoringResourceGroupName)
}

// Symbolic reference — Bicep infers the dependency automatically.
resource diagnosticSetting 'Microsoft.Insights/diagnosticSettings@2021-05-01-preview' = {
  name: 'send-to-law'
  scope: storageAccount
  properties: {
    workspaceId: logAnalytics.id  // No dependsOn needed. Bicep handles it.
  }
}
```

## Module Design

Modules are your unit of reuse and your unit of reasoning. Get the boundaries right and everything else gets easier.

- **One module = one logical unit of infrastructure.** A storage account and its private endpoint? One module. A storage account and a completely unrelated app service? Two modules.
- **Modules should be self-documenting** through their parameter and output descriptions. If someone has to open the module file to understand how to call it, the descriptions are incomplete.
- **Keep modules focused.** A module that deploys a storage account, a key vault, three managed identities, and a cosmos database is not a module — it's a monolith with a `.bicep` extension.
- **Pass only what's needed.** Don't forward your entire parameter set into a module. If the module needs five values, pass five values. This makes the contract explicit and prevents accidental coupling.
- **Outputs should expose what callers actually need.** Resource IDs, names, and principal IDs are common. Don't output everything just because you can.

## Tags

Every taggable resource gets tags. Every single one. Use a common tags variable to keep it consistent across the deployment.

```bicep
var commonTags = {
  Environment: environment
  Project: project
  ManagedBy: 'Bicep'
  LastDeployed: utcNow('yyyy-MM-dd')
}

// For resources that need additional tags, union them.
var storageSpecificTags = union(commonTags, {
  DataClassification: 'Internal'
  CostCenter: costCenterCode
})
```

Tags are how you find things, bill things, and audit things. A resource without tags is a resource that's going to cause someone a headache.

## Things That Will Get Your PR Rejected

This isn't a suggestion list. These are hard stops. If any of these show up in a pull request, it goes back for revision.

- **Parameters without `@description()`.** Every. Single. One. Gets. A. Description.
- **Hardcoded resource names or locations.** If it's not a parameter or derived from a parameter, it doesn't belong.
- **Secrets in outputs.** Never. If you need to pass a secret downstream, use Key Vault references or `@secure()` on the receiving parameter.
- **No tags on taggable resources.** If the resource supports tags and you didn't include them, that's a rejection.
- **Using `dependsOn` when a symbolic reference would work.** Explicit `dependsOn` is a code smell. It means either you're not using symbolic references or there's a genuine ordering issue — and if it's the latter, there better be a comment explaining it.
- **Deeply nested ternary expressions.** If your conditional logic needs more than one level, use a variable. `condition ? valueA : (otherCondition ? valueB : valueC)` is a debugging nightmare. Break it up.
- **No `metadata description` on the file.** Every file says what it is and why it exists. At the top. Before anything else.
- **Magic strings or numbers without explanation.** If there's a `256` or `'P1v3'` sitting in your code with no comment or named variable, nobody knows if that's intentional or a typo. Name it. Comment it. Make the intent clear.
