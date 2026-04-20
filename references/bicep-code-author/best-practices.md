# Best Practices

## Security Defaults
Every resource should start secure. Relax only with justification.

### Storage Accounts
```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: storageAccountName
  location: location
  tags: commonTags
  sku: { name: skuName }
  kind: 'StorageV2'
  properties: {
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
    supportsHttpsTrafficOnly: true
    defaultToOAuthAuthentication: true
    allowSharedKeyAccess: false          // Force Entra ID auth
    networkAcls: {
      defaultAction: 'Deny'             // Deny by default
      bypass: 'AzureServices'
    }
  }
}
```

### Key Vault
```bicep
resource keyVault 'Microsoft.KeyVault/vaults@2023-07-01' = {
  name: keyVaultName
  location: location
  tags: commonTags
  properties: {
    tenantId: tenant().tenantId
    sku: { family: 'A', name: 'standard' }
    enableRbacAuthorization: true        // RBAC over access policies
    enablePurgeProtection: true          // Cannot be disabled once enabled
    enableSoftDelete: true
    softDeleteRetentionInDays: 90
    networkAcls: {
      defaultAction: 'Deny'
      bypass: 'AzureServices'
    }
  }
}
```

### App Service
```bicep
resource webApp 'Microsoft.Web/sites@2023-12-01' = {
  name: appName
  location: location
  tags: commonTags
  kind: 'app,linux'
  identity: {
    type: 'UserAssigned'
    userAssignedIdentities: {
      '${managedIdentity.id}': {}
    }
  }
  properties: {
    serverFarmId: appServicePlan.id
    httpsOnly: true                      // Force HTTPS
    clientAffinityEnabled: false         // Unless you need sticky sessions
    siteConfig: {
      minTlsVersion: '1.2'
      ftpsState: 'Disabled'             // No FTP
      http20Enabled: true
      alwaysOn: true                     // Prevent cold starts (not on Free/Shared)
      healthCheckPath: '/health'
    }
  }
}
```

## Managed Identity Patterns
```bicep
// Prefer user-assigned — lifecycle independent from resource
resource managedIdentity 'Microsoft.ManagedIdentity/userAssignedIdentities@2023-01-31' = {
  name: '${namePrefix}-id'
  location: location
  tags: commonTags
}

// RBAC assignment — least privilege, scoped to resource
resource roleAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(storageAccount.id, managedIdentity.id, 'Storage Blob Data Reader')
  scope: storageAccount
  properties: {
    roleDefinitionId: subscriptionResourceId(
      'Microsoft.Authorization/roleDefinitions'
      '2a2b9908-6ea1-4ae2-8e65-a410df84e7d1'    // Storage Blob Data Reader
    )
    principalId: managedIdentity.properties.principalId
    principalType: 'ServicePrincipal'
  }
}
```

## Private Endpoint Pattern
```bicep
resource privateEndpoint 'Microsoft.Network/privateEndpoints@2024-05-01' = {
  name: '${namePrefix}-pe-kv'
  location: location
  tags: commonTags
  properties: {
    subnet: {
      id: privateEndpointSubnet.id
    }
    privateLinkServiceConnections: [
      {
        name: '${namePrefix}-plsc-kv'
        properties: {
          privateLinkServiceId: keyVault.id
          groupIds: ['vault']
        }
      }
    ]
  }
}

resource privateDnsZoneGroup 'Microsoft.Network/privateEndpoints/privateDnsZoneGroups@2024-05-01' = {
  parent: privateEndpoint
  name: 'default'
  properties: {
    privateDnsZoneConfigs: [
      {
        name: 'config'
        properties: {
          privateDnsZoneId: privateDnsZone.id
        }
      }
    ]
  }
}
```

## Diagnostic Settings Pattern
```bicep
// Apply to every resource that supports diagnostics
resource diagnosticSettings 'Microsoft.Insights/diagnosticSettings@2021-05-01-preview' = {
  name: 'send-to-law'
  scope: storageAccount
  properties: {
    workspaceId: logAnalyticsWorkspaceId
    logs: [
      { categoryGroup: 'allLogs', enabled: true }
    ]
    metrics: [
      { category: 'Transaction', enabled: true }
    ]
  }
}
```

## Resource Locks
```bicep
// Protect critical resources from accidental deletion
resource deleteLock 'Microsoft.Authorization/locks@2020-05-01' = {
  name: '${storageAccount.name}-lock'
  scope: storageAccount
  properties: {
    level: 'CanNotDelete'
    notes: 'Protected: production storage account with customer data.'
  }
}
```

## Common Tags Variable
```bicep
// Define once, use everywhere
var commonTags = {
  Environment: environment
  Project: project
  ManagedBy: 'Bicep'
  CostCenter: costCenter
  LastDeployed: utcNow('yyyy-MM-dd')    // Updated on every deployment
}

// Merge with resource-specific tags
resource rg 'Microsoft.Resources/resourceGroups@2024-03-01' = {
  name: rgName
  location: location
  tags: union(commonTags, {
    Purpose: 'Application hosting'
  })
}
```

## API Version Strategy
- Use the latest GA (Generally Available) API version
- Avoid preview API versions in production unless a feature requires it
- The `use-recent-api-versions` linter rule flags versions older than 2 years
- Update API versions during quarterly maintenance, not mid-sprint

## Naming Convention Helper
```bicep
// User-defined function for consistent naming
func resourceName(project string, environment string, resourceType string, instance string) string =>
  '${project}-${environment}-${resourceType}${!empty(instance) ? '-${instance}' : ''}'

// Usage
var rgName = resourceName(project, environment, 'rg', '')
var appName = resourceName(project, environment, 'app', 'api')
var kvName = resourceName(project, environment, 'kv', '')
```

## Project Structure (Recommended)
```
project/
├── bicepconfig.json              # Linter config + registry aliases
├── main.bicep                    # Orchestrator — calls modules
├── parameters/
│   ├── base.bicepparam           # Shared defaults
│   ├── dev.bicepparam
│   ├── staging.bicepparam
│   └── prod.bicepparam
├── modules/
│   ├── networking.bicep          # VNet + subnets + NSGs
│   ├── storage-account.bicep     # Storage with security defaults
│   ├── key-vault.bicep           # Key Vault with RBAC
│   ├── app-service.bicep         # App Service + plan
│   ├── managed-identity.bicep    # Identity + role assignments
│   └── monitoring.bicep          # Diagnostics + alerts
└── scripts/
    ├── deploy.sh                 # Deployment wrapper
    └── validate.sh               # Build + validate + what-if
```
