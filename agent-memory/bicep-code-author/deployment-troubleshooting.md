# Deployment Troubleshooting

Common Bicep deployment errors, their causes, and fixes.

## Error: ResourceNotFound
**Message:** `The Resource 'Microsoft.X/y/z' under resource group 'rg-name' was not found.`
**Cause:** Referencing a resource with `existing` that doesn't exist, or dependency timing issue.
**Fix:**
- Verify the resource name and resource group are correct
- Check for typos in the `scope:` property
- Ensure the resource was created in a prior deployment or exists in the target subscription
```bicep
// Confirm scope is correct
resource kv 'Microsoft.KeyVault/vaults@2023-07-01' existing = {
  name: keyVaultName
  scope: resourceGroup(kvResourceGroupName)  // Double-check this
}
```

## Error: InvalidTemplate — Circular Dependency
**Message:** `Circular dependency detected: resource1 -> resource2 -> resource1`
**Fix:** Break the cycle by:
1. Removing unnecessary `dependsOn` entries (prefer implicit dependencies)
2. Moving configuration into child resources that deploy after both parents
3. Splitting into separate deployments if resources truly need each other's runtime properties

## Error: DeploymentFailed — Conflict (409)
**Message:** `The resource 'X' already exists in resource group 'Y'.`
**Cause:** Name collision — another resource with the same name exists, or a previous deployment is still running.
**Fix:**
- Use `uniqueString()` for globally unique names (storage accounts, Key Vaults)
- Check for in-flight deployments: `az deployment group list --resource-group rg-name --query "[?properties.provisioningState=='Running']"`
- For soft-deleted resources (Key Vault, Cognitive Services): purge first or use a different name

## Error: InvalidTemplateDeployment — Quota Exceeded
**Message:** `Operation could not be completed as it results in exceeding approved quota.`
**Fix:**
- Check current usage: `az vm list-usage --location eastus2 --output table`
- Request quota increase via Azure Portal > Subscriptions > Usage + quotas
- Consider deploying to a different region or using a smaller SKU for non-prod

## Error: AuthorizationFailed
**Message:** `The client 'X' does not have authorization to perform action 'Y' over scope 'Z'.`
**Fix:**
- Verify the deploying identity has the correct RBAC role at the correct scope
- For subscription-scoped deployments, the identity needs Contributor at subscription level
- For management group scope, Owner or custom role at the management group
- Service principals need explicit role assignments — they don't inherit from user group membership

## Error: InvalidResourceReference
**Message:** `The reference to resource 'X' could not be resolved.`
**Cause:** Using a symbolic reference to a conditional resource without guarding it.
**Fix:**
```bicep
module sql './modules/sql.bicep' = if (deploySQL) { ... }
// Guard all references to conditional resources
output fqdn string = deploySQL ? sql.outputs.fqdn : ''
```

## Error: LinkedInvalidPropertyValue — API Version Issue
**Message:** `The property 'someNewProperty' is not valid for API version '2021-01-01'.`
**Fix:** Update the API version to a newer GA version that supports the property:
```bicep
// Old API — missing properties
resource kv 'Microsoft.KeyVault/vaults@2021-01-01' = { ... }
// Updated API — supports new properties
resource kv 'Microsoft.KeyVault/vaults@2023-07-01' = { ... }
```

## Error: ExceededMaxTemplateDepth
**Message:** `The template nesting depth exceeds the maximum of 5.`
**Fix:** Flatten module hierarchy. ARM allows a maximum nesting depth of 5 levels. If you have module A calling module B calling module C calling module D calling module E, you're at the limit.

## Error: DeploymentOutputSizeExceeded
**Message:** `The deployment output size exceeds the limit of 4 MB.`
**Fix:** Reduce output verbosity. Don't output entire resource objects — output only the IDs, names, and endpoints you need.

## Debugging Techniques
### View Deployment Operations
```bash
# List all operations in a failed deployment
az deployment group show --resource-group rg-name --name deployment-name \
  --query "properties.error"

# Get detailed operation-level errors
az deployment operation group list --resource-group rg-name --name deployment-name \
  --query "[?properties.provisioningState=='Failed']"
```

### Inspect Compiled ARM Template
```bash
# Build to see the actual ARM JSON that gets deployed
bicep build main.bicep --outfile main.json

# Check compiled template size (4 MB limit)
(Get-Item main.json).Length / 1MB
```

### Validate Before Deploying
```bash
# Syntax and schema validation — catches most template errors
az deployment group validate --resource-group rg-name \
  --template-file main.bicep --parameters main.bicepparam

# What-if preview — catches runtime issues like naming conflicts
az deployment group what-if --resource-group rg-name \
  --template-file main.bicep --parameters main.bicepparam
```
