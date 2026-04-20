# Deployment Stacks

Deployment stacks manage a collection of Azure resources as a single unit. They track which resources belong to a deployment and can protect them from deletion or modification.

## Creating a Stack
```bash
# Resource group scope
az stack group create \
  --name myapp-prod-stack \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters parameters/prod.bicepparam \
  --action-on-unmanage deleteResources \
  --deny-settings-mode denyDelete

# Subscription scope
az stack sub create \
  --name myapp-platform-stack \
  --location eastus2 \
  --template-file platform.bicep \
  --parameters parameters/platform.bicepparam \
  --action-on-unmanage deleteResources \
  --deny-settings-mode denyWriteAndDelete

# Management group scope
az stack mg create \
  --name policy-stack \
  --management-group-id myMgGroup \
  --location eastus2 \
  --template-file policies.bicep \
  --action-on-unmanage detachResources \
  --deny-settings-mode none
```

## Action on Unmanage
What happens to resources removed from the template on the next update:

| Setting | Behavior |
|---|---|
| `deleteResources` | Deletes resources removed from template. Resource groups and management groups are detached. |
| `deleteAll` | Deletes resources AND resource groups removed from template. |
| `detachResources` | Detaches resources (they continue to exist but are no longer tracked by the stack). |

```bash
# Safest for production — delete unmanaged resources
az stack group create \
  --name prod-stack \
  --resource-group rg-prod \
  --template-file main.bicep \
  --action-on-unmanage deleteResources

# For development — detach so you can experiment
az stack group create \
  --name dev-stack \
  --resource-group rg-dev \
  --template-file main.bicep \
  --action-on-unmanage detachResources
```

## Deny Settings
Prevent out-of-band modifications to stack-managed resources:

| Mode | Effect |
|---|---|
| `none` | No deny settings — anyone with RBAC can modify resources |
| `denyDelete` | Prevents deletion of managed resources outside the stack |
| `denyWriteAndDelete` | Prevents both modification and deletion outside the stack |

```bash
# Production: prevent deletion
az stack group create \
  --name prod-stack \
  --resource-group rg-prod \
  --template-file main.bicep \
  --deny-settings-mode denyDelete \
  --action-on-unmanage deleteResources

# Critical infrastructure: prevent all modifications
az stack group create \
  --name core-infra-stack \
  --resource-group rg-core \
  --template-file core.bicep \
  --deny-settings-mode denyWriteAndDelete \
  --action-on-unmanage deleteResources

# Exclude specific principals from deny settings (break-glass)
az stack group create \
  --name prod-stack \
  --resource-group rg-prod \
  --template-file main.bicep \
  --deny-settings-mode denyWriteAndDelete \
  --deny-settings-excluded-principals <object-id-1> <object-id-2> \
  --action-on-unmanage deleteResources

# Exclude specific actions from deny settings
az stack group create \
  --name prod-stack \
  --resource-group rg-prod \
  --template-file main.bicep \
  --deny-settings-mode denyWriteAndDelete \
  --deny-settings-excluded-actions Microsoft.Compute/virtualMachines/restart/action \
  --action-on-unmanage deleteResources
```

## Updating a Stack
```bash
# Update with new template — will add/modify/remove resources
az stack group create \
  --name myapp-prod-stack \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters parameters/prod.bicepparam \
  --action-on-unmanage deleteResources \
  --deny-settings-mode denyDelete

# Same command as create — idempotent
# If the stack exists, it updates. If not, it creates.
```

## Listing and Inspecting Stacks
```bash
# List stacks in a resource group
az stack group list --resource-group rg-myapp-prod

# Show stack details including managed resources
az stack group show \
  --name myapp-prod-stack \
  --resource-group rg-myapp-prod

# List resources managed by a stack
az stack group show \
  --name myapp-prod-stack \
  --resource-group rg-myapp-prod \
  --query "resources[].id" -o tsv
```

## Deleting a Stack
```bash
# Delete stack AND managed resources
az stack group delete \
  --name myapp-prod-stack \
  --resource-group rg-myapp-prod \
  --action-on-unmanage deleteResources \
  --yes

# Delete stack but KEEP resources (detach)
az stack group delete \
  --name myapp-prod-stack \
  --resource-group rg-myapp-prod \
  --action-on-unmanage detachResources \
  --yes
```

## Stack vs Traditional Deployment

| Feature | Traditional `az deployment` | Deployment Stacks |
|---|---|---|
| Resource tracking | No — fire and forget | Yes — knows what it manages |
| Drift protection | No | Yes — deny settings |
| Orphan cleanup | Manual | Automatic (actionOnUnmanage) |
| Incremental updates | Adds, doesn't remove | Adds, modifies, AND removes |
| Complete mode risk | Deletes EVERYTHING not in template | Only removes resources that were previously managed |

## CI/CD Integration
```yaml
# Azure Pipelines
- task: AzureCLI@2
  displayName: 'Deploy Stack'
  inputs:
    azureSubscription: 'production-wif'
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      az stack group create \
        --name $(stackName) \
        --resource-group $(resourceGroup) \
        --template-file main.bicep \
        --parameters parameters/$(environment).bicepparam \
        --action-on-unmanage deleteResources \
        --deny-settings-mode denyDelete \
        --yes
```

```yaml
# GitHub Actions
- name: Deploy Stack
  uses: azure/cli@v2
  with:
    inlineScript: |
      az stack group create \
        --name ${{ env.STACK_NAME }} \
        --resource-group ${{ env.RESOURCE_GROUP }} \
        --template-file main.bicep \
        --parameters parameters/${{ env.ENVIRONMENT }}.bicepparam \
        --action-on-unmanage deleteResources \
        --deny-settings-mode denyDelete \
        --yes
```
