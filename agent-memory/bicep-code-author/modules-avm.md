# Modules & Azure Verified Modules

## Local Modules
```bicep
// Consuming a local module
module storageAccount './modules/storage-account.bicep' = {
  name: 'deploy-storage-${environment}'    // Deployment name (must be unique per scope)
  params: {
    storageAccountName: storageName
    location: location
    tags: commonTags
    skuName: environment == 'prod' ? 'Standard_GRS' : 'Standard_LRS'
  }
}

// Access module outputs
output storageId string = storageAccount.outputs.storageAccountId
output storagePrimaryEndpoint string = storageAccount.outputs.primaryBlobEndpoint
```

### Module File Structure
```
project/
├── main.bicep                    # Orchestrator
├── main.bicepparam               # Parameter values
└── modules/
    ├── storage-account.bicep     # Storage module
    ├── key-vault.bicep           # Key Vault module
    ├── networking.bicep          # VNet + subnets
    └── app-service.bicep         # App Service + plan
```

### Writing a Module
```bicep
// modules/storage-account.bicep
metadata description = 'Deploys a Storage Account with standard security configuration.'

// --- Parameters ---
@description('Globally unique storage account name.')
@minLength(3)
@maxLength(24)
param storageAccountName string

@description('Azure region.')
param location string

@description('Resource tags.')
param tags object

@description('Storage SKU name.')
@allowed(['Standard_LRS', 'Standard_GRS', 'Standard_ZRS', 'Standard_GZRS'])
param skuName string = 'Standard_LRS'

@description('Enable blob versioning.')
param enableVersioning bool = true

@description('Minimum TLS version.')
param minimumTlsVersion string = 'TLS1_2'

// --- Resources ---
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: storageAccountName
  location: location
  tags: tags
  sku: { name: skuName }
  kind: 'StorageV2'
  properties: {
    minimumTlsVersion: minimumTlsVersion
    allowBlobPublicAccess: false
    supportsHttpsTrafficOnly: true
    defaultToOAuthAuthentication: true
    allowSharedKeyAccess: false
    networkAcls: {
      defaultAction: 'Deny'
      bypass: 'AzureServices'
    }
  }
}

resource blobServices 'Microsoft.Storage/storageAccounts/blobServices@2023-05-01' = {
  parent: storageAccount
  name: 'default'
  properties: {
    isVersioningEnabled: enableVersioning
    deleteRetentionPolicy: {
      enabled: true
      days: 7
    }
  }
}

// --- Outputs ---
@description('Resource ID of the storage account.')
output storageAccountId string = storageAccount.id

@description('Name of the storage account.')
output storageAccountName string = storageAccount.name

@description('Primary blob endpoint.')
output primaryBlobEndpoint string = storageAccount.properties.primaryEndpoints.blob

@description('Principal ID of the system-assigned managed identity (if enabled).')
output principalId string = storageAccount.identity.?principalId ?? ''
```

## Bicep Registry Modules
```bicep
// Public Bicep Registry (Microsoft-maintained)
module storageAccount 'br/public:avm/res/storage/storage-account:0.14.0' = {
  name: 'deploy-storage'
  params: {
    name: storageName
    location: location
    tags: commonTags
    skuName: 'Standard_LRS'
  }
}

// Private registry (ACR)
module customModule 'br:myregistry.azurecr.io/bicep/modules/custom-app:1.0.0' = {
  name: 'deploy-custom-app'
  params: {
    // ...
  }
}
```

### Configure Registry Aliases (bicepconfig.json)
```json
{
  "moduleAliases": {
    "br": {
      "public": {
        "registry": "mcr.microsoft.com",
        "modulePath": "bicep"
      },
      "myregistry": {
        "registry": "myregistry.azurecr.io",
        "modulePath": "bicep/modules"
      }
    }
  }
}
```

## Template Specs
```bicep
// Consume a template spec as a module
module networkFromSpec 'ts:mySubscription/networking-rg/vnet-standard:1.0' = {
  name: 'deploy-vnet-from-spec'
  params: {
    vnetName: 'vnet-prod-01'
    addressSpace: '10.0.0.0/16'
  }
}

// Publish a template spec
// az ts create --name vnet-standard --version 1.0 --resource-group networking-rg --template-file networking.bicep
```

## Azure Verified Modules (AVM) — Reference Only
AVM is the official Microsoft module library. **Reference AVM for patterns and conventions, but write your own modules.** AVM modules are over-engineered for most use cases — they're excellent as reference implementations to study, but they inflate compiled template size, introduce unnecessary complexity, and make you dependent on Microsoft's release cadence. Write lean, purpose-built modules that include only what you need.

### Why Reference-Only
- AVM modules include every possible feature for a resource type, inflating compiled ARM JSON (often 10-20KB per module reference)
- You can't strip unused features — they're always compiled into the template
- Version updates may introduce breaking changes or unwanted defaults
- Debugging issues inside AVM modules requires reading generated ARM JSON
- **Study AVM for:** Security defaults, interface patterns, type definitions, naming conventions
- **Write your own for:** Production deployments, lean templates, full control

### Finding AVM for Reference
- Browse: https://aka.ms/AVM/bicep/modules
- Registry: `br/public:avm/res/<provider>/<resource-type>:<version>`
- Pattern types:
  - `avm/res/` — Resource modules (single Azure resource + children)
  - `avm/ptn/` — Pattern modules (multi-resource patterns like landing zones)

### AVM Resource Module Examples (Reference Patterns)
```bicep
// Study these patterns for security defaults and interface design — then write your own

// Key Vault with RBAC
module keyVault 'br/public:avm/res/key-vault/vault:0.9.0' = {
  name: 'deploy-key-vault'
  params: {
    name: kvName
    location: location
    tags: commonTags
    enableRbacAuthorization: true
    enablePurgeProtection: true
    softDeleteRetentionInDays: 90
    roleAssignments: [
      {
        roleDefinitionIdOrName: 'Key Vault Secrets User'
        principalId: appIdentity.outputs.principalId
        principalType: 'ServicePrincipal'
      }
    ]
    secrets: [
      {
        name: 'connection-string'
        value: sqlModule.outputs.connectionString
      }
    ]
    diagnosticSettings: [
      {
        workspaceResourceId: logAnalyticsWorkspaceId
      }
    ]
  }
}

// Virtual Network
module vnet 'br/public:avm/res/network/virtual-network:0.5.0' = {
  name: 'deploy-vnet'
  params: {
    name: vnetName
    location: location
    tags: commonTags
    addressPrefixes: ['10.0.0.0/16']
    subnets: [
      {
        name: 'snet-app'
        addressPrefix: '10.0.1.0/24'
        delegation: 'Microsoft.Web/serverFarms'
      }
      {
        name: 'snet-pe'
        addressPrefix: '10.0.2.0/24'
      }
    ]
  }
}

// User-Assigned Managed Identity
module identity 'br/public:avm/res/managed-identity/user-assigned-identity:0.4.0' = {
  name: 'deploy-identity'
  params: {
    name: '${namePrefix}-id'
    location: location
    tags: commonTags
  }
}
```

### AVM Pattern Module Example (Reference Only)
```bicep
// Study pattern modules for multi-resource orchestration patterns
// Landing zone pattern — deploys VNet, NSGs, route tables, peering
module landingZone 'br/public:avm/ptn/network/hub-networking:0.3.0' = {
  name: 'deploy-hub-network'
  params: {
    location: location
    hubVirtualNetworkAddressPrefix: '10.0.0.0/16'
    // ... pattern-specific params
  }
}
```

## Module Loops
```bicep
// Deploy multiple instances from a configuration array
param webApps array = [
  { name: 'frontend', sku: 'B1', runtime: 'NODE|20-lts' }
  { name: 'api', sku: 'S1', runtime: 'DOTNETCORE|8.0' }
]

module apps './modules/app-service.bicep' = [for app in webApps: {
  name: 'deploy-app-${app.name}'
  params: {
    appName: '${namePrefix}-${app.name}'
    location: location
    tags: commonTags
    skuName: app.sku
    linuxFxVersion: app.runtime
  }
}]

// Access outputs from looped modules
output appUrls array = [for (app, i) in webApps: {
  name: app.name
  url: apps[i].outputs.defaultHostName
}]
```

## Module Dependencies
```bicep
// Implicit — Bicep resolves from parameter references
module app './modules/app-service.bicep' = {
  params: {
    subnetId: networking.outputs.appSubnetId    // Implicit dependency on networking
  }
}

// Explicit — when there's no data dependency but order matters
module rbac './modules/rbac.bicep' = {
  name: 'deploy-rbac'
  dependsOn: [
    keyVault    // Must exist before role assignments
  ]
  params: {
    keyVaultId: keyVault.outputs.resourceId
    principalId: identity.outputs.principalId
  }
}
```
