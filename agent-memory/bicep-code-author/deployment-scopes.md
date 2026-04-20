# Deployment Scopes

## Scope Hierarchy
```
Tenant
└── Management Group
    └── Subscription
        └── Resource Group (default)
```

## Resource Group Scope (Default)
No `targetScope` needed — this is the default.
```bicep
// main.bicep — deploys to a resource group
@description('Azure region for resources.')
param location string = resourceGroup().location

@description('Project name used in resource naming.')
param project string

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: '${project}st${uniqueString(resourceGroup().id)}'
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
  tags: { Project: project }
}
```

Deploy:
```bash
az deployment group create \
  --resource-group rg-myproject-dev \
  --template-file main.bicep \
  --parameters project='myproject'
```

## Subscription Scope
```bicep
targetScope = 'subscription'

@description('Azure region for the resource group.')
param location string

@description('Environment name.')
@allowed(['dev', 'staging', 'prod'])
param environment string

// Create the resource group
resource rg 'Microsoft.Resources/resourceGroups@2024-03-01' = {
  name: 'rg-myproject-${environment}'
  location: location
  tags: { Environment: environment }
}

// Deploy into the resource group using a module
module resources './modules/resources.bicep' = {
  name: 'deploy-resources-${environment}'
  scope: rg    // Target the resource group we just created
  params: {
    location: location
    environment: environment
  }
}
```

Deploy:
```bash
az deployment sub create \
  --location eastus2 \
  --template-file main.bicep \
  --parameters environment='dev' location='eastus2'
```

## Management Group Scope
```bicep
targetScope = 'managementGroup'

@description('Policy display name.')
param policyName string

@description('Allowed locations for resources.')
param allowedLocations string[]

// Policy definition at management group level
resource policyDef 'Microsoft.Authorization/policyDefinitions@2021-06-01' = {
  name: guid(policyName, managementGroup().id)
  properties: {
    policyType: 'Custom'
    mode: 'Indexed'
    displayName: policyName
    description: 'Restrict resource locations to approved regions.'
    parameters: {}
    policyRule: {
      if: {
        not: {
          field: 'location'
          in: allowedLocations
        }
      }
      then: {
        effect: 'deny'
      }
    }
  }
}

// Policy assignment
resource policyAssignment 'Microsoft.Authorization/policyAssignments@2024-04-01' = {
  name: guid(policyName, 'assignment', managementGroup().id)
  properties: {
    policyDefinitionId: policyDef.id
    displayName: '${policyName} Assignment'
    enforcementMode: 'Default'
  }
}
```

Deploy:
```bash
az deployment mg create \
  --management-group-id myManagementGroup \
  --location eastus2 \
  --template-file policy.bicep \
  --parameters policyName='Restrict Locations' allowedLocations='["eastus2","westus2"]'
```

## Tenant Scope
```bicep
targetScope = 'tenant'

@description('Management group display name.')
param mgName string

@description('Parent management group ID.')
param parentMgId string = ''

// Create management group hierarchy
resource mg 'Microsoft.Management/managementGroups@2021-04-01' = {
  name: mgName
  properties: {
    displayName: mgName
    details: !empty(parentMgId) ? {
      parent: { id: tenantResourceId('Microsoft.Management/managementGroups', parentMgId) }
    } : null
  }
}
```

Deploy:
```bash
az deployment tenant create \
  --location eastus2 \
  --template-file hierarchy.bicep \
  --parameters mgName='Platform'
```

## Cross-Scope Deployments (Module Scoping)
The real power — deploying from one scope into another using modules.

```bicep
targetScope = 'subscription'

param location string
param environment string

// Create the resource group at subscription scope
resource rg 'Microsoft.Resources/resourceGroups@2024-03-01' = {
  name: 'rg-network-${environment}'
  location: location
}

// Deploy networking into that resource group
module networking './modules/networking.bicep' = {
  name: 'deploy-networking'
  scope: rg
  params: {
    location: location
    environment: environment
  }
}

// Deploy into a DIFFERENT subscription
module crossSubDeploy './modules/monitoring.bicep' = {
  name: 'deploy-monitoring'
  scope: resourceGroup('other-subscription-id', 'rg-monitoring')
  params: {
    location: location
  }
}

// Deploy at management group scope from subscription scope
module mgPolicy './modules/policy.bicep' = {
  name: 'deploy-policy'
  scope: managementGroup('myMgGroup')
  params: {
    allowedLocations: ['eastus2', 'westus2']
  }
}

// Deploy at tenant scope
module tenantRbac './modules/tenant-rbac.bicep' = {
  name: 'deploy-tenant-rbac'
  scope: tenant()
  params: {
    principalId: servicePrincipalId
  }
}
```

## Extension Resources (Scope to a Resource)
```bicep
// Lock a resource — extension resource scoped to the storage account
resource storageLock 'Microsoft.Authorization/locks@2020-05-01' = {
  name: 'storage-delete-lock'
  scope: storageAccount    // Scoped to a specific resource
  properties: {
    level: 'CanNotDelete'
    notes: 'Protect production storage from accidental deletion.'
  }
}

// Role assignment scoped to a resource
resource roleAssignment 'Microsoft.Authorization/roleAssignments@2022-04-01' = {
  name: guid(storageAccount.id, principalId, 'Storage Blob Data Reader')
  scope: storageAccount
  properties: {
    roleDefinitionId: subscriptionResourceId('Microsoft.Authorization/roleDefinitions', '2a2b9908-6ea1-4ae2-8e65-a410df84e7d1')
    principalId: principalId
    principalType: 'ServicePrincipal'
  }
}

// Diagnostic settings scoped to a resource
resource diagnostics 'Microsoft.Insights/diagnosticSettings@2021-05-01-preview' = {
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

## Scope Functions Reference
| Function | Returns | Use When |
|---|---|---|
| `resourceGroup()` | Current RG | Default scope |
| `resourceGroup('name')` | Named RG (same sub) | Cross-RG in same subscription |
| `resourceGroup('subId', 'name')` | Named RG (different sub) | Cross-subscription |
| `subscription()` | Current subscription | Subscription-scoped resources |
| `subscription('subId')` | Different subscription | Cross-subscription |
| `managementGroup('id')` | Management group | Policy/governance deployments |
| `tenant()` | Tenant root | Management group hierarchy |
