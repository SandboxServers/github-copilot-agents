# State & Deployment Model

## ARM Deployment Model
Bicep has NO local state file (unlike Terraform). Bicep transpiles to ARM JSON, and Azure Resource Manager is the state.

### How It Works
1. Bicep file → transpiled to ARM template JSON
2. ARM template submitted to Azure Resource Manager
3. ARM compares desired state against current Azure state
4. ARM creates/updates/deletes resources to match

### Implications
- No `.tfstate` file to manage, lock, or protect
- Azure IS the state — what exists in Azure is the truth
- No import needed — if a resource exists, just reference it with `existing`
- Deployments are recorded in the resource group's deployment history

## Deployment Modes

### Incremental (Default — Use This)
- Adds or updates resources in the template
- Does NOT touch resources that exist in Azure but aren't in the template
- Safe — won't accidentally delete anything

```bash
az deployment group create \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters parameters/prod.bicepparam \
  --mode Incremental    # This is the default, explicit for clarity
```

### Complete (Dangerous — Avoid)
- Deletes any resource in the resource group NOT in the template
- Will destroy resources managed by other teams or deployments
- The only valid use case: a resource group that is EXCLUSIVELY managed by this one template

```bash
# DANGEROUS — only use if you own the entire resource group
az deployment group create \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters parameters/prod.bicepparam \
  --mode Complete

# ALWAYS run what-if first with complete mode
az deployment group what-if \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters parameters/prod.bicepparam \
  --mode Complete
```

### Why Deployment Stacks Replace Complete Mode
Use deployment stacks instead of complete mode. Stacks provide:
- Tracking of managed resources (like complete mode)
- Protection via deny settings (better than complete mode)
- No risk of deleting unrelated resources
- Works across resource group boundaries

## Deployment History
Azure keeps a deployment history per scope. The limit is 800 deployments per resource group.

```bash
# List deployments
az deployment group list \
  --resource-group rg-myapp-prod \
  --query "[].{name:name, state:properties.provisioningState, timestamp:properties.timestamp}" \
  -o table

# Show a specific deployment
az deployment group show \
  --resource-group rg-myapp-prod \
  --name deploy-storage-prod

# Delete old deployments (auto-cleanup happens at 800)
az deployment group delete \
  --resource-group rg-myapp-prod \
  --name old-deployment-name
```

## Referencing Existing Resources
Since Azure is the state, you reference existing resources directly:

```bicep
// Reference a resource in the same resource group
resource existingVnet 'Microsoft.Network/virtualNetworks@2024-05-01' existing = {
  name: 'vnet-prod-01'
}

// Reference a resource in a different resource group
resource existingKeyVault 'Microsoft.KeyVault/vaults@2023-07-01' existing = {
  name: 'kv-shared-prod'
  scope: resourceGroup('rg-shared-prod')
}

// Reference a resource in a different subscription
resource existingLogAnalytics 'Microsoft.OperationalInsights/workspaces@2023-09-01' existing = {
  name: 'law-central'
  scope: resourceGroup('other-sub-id', 'rg-monitoring')
}

// Use existing resource properties
output vnetId string = existingVnet.id
output kvUri string = existingKeyVault.properties.vaultUri
```

## Deployment Outputs and Chaining
```bash
# Get outputs from a completed deployment
az deployment group show \
  --resource-group rg-myapp-prod \
  --name deploy-storage-prod \
  --query "properties.outputs.storageAccountId.value" -o tsv

# Use in subsequent deployments or scripts
storageId=$(az deployment group show \
  --resource-group rg-myapp-prod \
  --name deploy-storage-prod \
  --query "properties.outputs.storageAccountId.value" -o tsv)

az deployment group create \
  --resource-group rg-myapp-prod \
  --template-file app.bicep \
  --parameters storageAccountId=$storageId
```

## Concurrency and Locking
- ARM processes resources in parallel by default (based on dependency graph)
- Resource group-level locking: use `Microsoft.Authorization/locks`
- No built-in deployment locking — use CI/CD concurrency controls
- Deployment stacks with deny settings provide runtime protection
