# Resource Adoption

Bringing existing Azure resources under Terraform management. Use import blocks (Terraform 1.5+), not the CLI `terraform import` command.

## Import Block Pattern
```hcl
# Step 1: Write the import block pointing to the Azure resource ID
import {
  to = azurerm_resource_group.main
  id = "/subscriptions/xxxx/resourceGroups/rg-myapp-prod"
}

# Step 2: Write a matching resource block (or generate one)
resource "azurerm_resource_group" "main" {
  name     = "rg-myapp-prod"
  location = "eastus2"
  tags     = local.common_tags
}

# Step 3: Run terraform plan — it should show "will be imported" with no changes
# Step 4: Run terraform apply to import into state
# Step 5: Remove the import block after successful import
```

## Config Generation (Terraform 1.5+)
When adopting many resources, generate config stubs instead of writing them by hand:
```bash
# Generate HCL config for the imported resource
terraform plan -generate-config-out=generated.tf
```
This creates `generated.tf` with a resource block matching the imported state. Review and refactor the generated code — it's verbose and needs cleanup (remove computed attributes, parameterize values, apply naming conventions).

## Bulk Import Workflow
For adopting an entire environment:
1. **Inventory** — List resources with `az resource list --resource-group rg-target -o table`
2. **Prioritize** — Import in dependency order: resource groups → networks → compute → data
3. **Write import blocks** — One per resource, all in an `imports.tf` file
4. **Generate config** — Use `-generate-config-out` for the first pass
5. **Refactor** — Move generated code into proper module structure, parameterize
6. **Plan** — Verify zero diff after import (`No changes. Infrastructure is up-to-date.`)
7. **Clean up** — Remove `imports.tf` and `generated.tf` after successful apply

## Importing into `for_each` Resources
```hcl
import {
  to = azurerm_subnet.this["app-gateway"]
  id = "/subscriptions/xxxx/resourceGroups/rg-network/providers/Microsoft.Network/virtualNetworks/vnet-hub/subnets/snet-app-gateway"
}

import {
  to = azurerm_subnet.this["workload"]
  id = "/subscriptions/xxxx/resourceGroups/rg-network/providers/Microsoft.Network/virtualNetworks/vnet-hub/subnets/snet-workload"
}
```
The `to` address must match the `for_each` key exactly.

## Importing into Modules
```hcl
import {
  to = module.key_vault.azurerm_key_vault.main
  id = "/subscriptions/xxxx/resourceGroups/rg-myapp/providers/Microsoft.KeyVault/vaults/kv-myapp-prod"
}
```
Use the full module path: `module.<name>.<resource_type>.<resource_name>`.

## Common Import Pitfalls
- **Drift after import** — The imported resource may have properties not in your config. Run `terraform plan` immediately to find mismatches and update your config to match.
- **Sensitive values** — Import doesn't capture secrets (passwords, keys). You must set them in config or mark them as ignored.
- **Provider features** — Some `azurerm` feature flags (e.g., `purge_soft_delete_on_destroy`) affect import behavior for Key Vaults and similar resources.
- **Read-only attributes** — Generated config may include computed attributes (`id`, `fqdn`). Remove these — they're outputs, not inputs.
- **State-only resources** — Some resources can be imported but not fully managed (e.g., Azure AD resources require the `azuread` provider, not `azurerm`).

## Import vs `terraform state mv`
| Scenario | Use |
|---|---|
| Adopt existing Azure resource | `import` block |
| Rename resource in config | `moved` block (see [code-patterns.md](code-patterns.md)) |
| Move resource between state files | `terraform state mv` |
| Split a monolith into multiple states | `terraform state mv` + new backend config |

## CLI Import (Legacy)
Still works but not recommended for new adoption:
```bash
terraform import azurerm_resource_group.main /subscriptions/xxxx/resourceGroups/rg-myapp-prod
```
Disadvantages: not declarative, not reviewable in PRs, doesn't generate config, must be run interactively.
