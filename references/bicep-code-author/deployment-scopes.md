## Deployment Scopes

### Target Scopes
| Scope | targetScope | CLI Command | Use Case |
|---|---|---|---|
| **Resource Group** | `resourceGroup` (default) | `az deployment group create` | Most resources |
| **Subscription** | `subscription` | `az deployment sub create` | Resource groups, policies, role assignments |
| **Management Group** | `managementGroup` | `az deployment mg create` | Policy definitions, management group structure |
| **Tenant** | `tenant` | `az deployment tenant create` | Management group hierarchy, tenant-level policies |

### Cross-Scope Deployments
Use modules to deploy at different scopes from the parent:
```bicep
// From subscription scope, deploy to a resource group
module rgDeployment 'resources.bicep' = {
  scope: resourceGroup('rg-name')
  name: 'rgDeploy'
  params: { ... }
}

// From resource group scope, deploy to another resource group
module crossRg 'other-resources.bicep' = {
  scope: resourceGroup('other-subscription-id', 'other-rg-name')
  name: 'crossRgDeploy'
  params: { ... }
}
```

**Scope functions:** `resourceGroup()`, `subscription()`, `managementGroup()`, `tenant()`
