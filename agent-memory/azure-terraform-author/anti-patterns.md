# Anti-Patterns

Common Terraform/Azure mistakes. If you see these in a PR, flag them.

## Hardcoding Values
```hcl
# BAD — environment, region, names baked in
resource "azurerm_resource_group" "main" {
  name     = "rg-myapp-prod"
  location = "eastus2"
}

# GOOD — parameterized
resource "azurerm_resource_group" "main" {
  name     = "rg-${var.project}-${var.environment}"
  location = var.location
}
```

## Not Using Data Sources for Existing Resources
```hcl
# BAD — hardcoded resource ID
subnet_id = "/subscriptions/xxx/resourceGroups/rg-network/providers/Microsoft.Network/virtualNetworks/vnet-hub/subnets/snet-app"

# GOOD — dynamic lookup
data "azurerm_subnet" "app" {
  name                 = "snet-app"
  virtual_network_name = "vnet-hub"
  resource_group_name  = "rg-network"
}
subnet_id = data.azurerm_subnet.app.id
```

## Monolithic Root Modules
Everything in one huge `main.tf` — hundreds of resources, no separation. Break into child modules by domain (networking, compute, data). A root module should orchestrate modules, not define resources directly.

## Not Pinning Provider Versions
```hcl
# BAD — no version constraint, any version installs
required_providers {
  azurerm = { source = "hashicorp/azurerm" }
}

# GOOD — pin to major, allow minor/patch
required_providers {
  azurerm = { source = "hashicorp/azurerm", version = "~> 4.0" }
}
```
Always commit `.terraform.lock.hcl` — it pins exact provider versions per platform.

## Local State (No Remote Backend)
Local `terraform.tfstate` means no locking, no team collaboration, no audit trail. Always use a remote backend (Azure Storage with `use_azuread_auth = true`). See [state-management.md](state-management.md).

## Missing `prevent_destroy` on Critical Resources
Databases, Key Vaults, storage accounts, and AKS clusters should have `lifecycle { prevent_destroy = true }`. Without it, a `terraform destroy` or refactor can delete production data.

## Applying Without Reviewing Plan Output
Never pipe `terraform plan` into `terraform apply` without review. In CI/CD, always use `terraform plan -out=tfplan` then `terraform apply tfplan` as separate steps with a gate between them.

## No Naming Convention
Inconsistent names (`rg-myapp`, `MyAppRG`, `resource-group-1`) create confusion. Adopt a convention and enforce it via variable validation:
```hcl
variable "name" {
  validation {
    condition     = can(regex("^[a-z0-9-]+$", var.name))
    error_message = "Name must be lowercase alphanumeric with hyphens."
  }
}
```

## Not Using `ignore_changes` for Externally-Managed Properties
Azure modifies certain properties outside Terraform (autoscaler node counts, policy-applied tags). Without `ignore_changes`, every plan shows drift. See [common-gotchas.md](common-gotchas.md) for the full list.

## `count` When `for_each` Is Clearer
```hcl
# BAD — count with list index; removing an item reindexes everything
resource "azurerm_storage_account" "this" {
  count = length(var.storage_accounts)
  name  = var.storage_accounts[count.index]
}

# GOOD — for_each with set/map; keyed by name
resource "azurerm_storage_account" "this" {
  for_each = toset(var.storage_accounts)
  name     = each.value
}
```
Reserve `count` for binary conditionals (`count = var.enable_feature ? 1 : 0`).

## Circular Module Dependencies
Module A outputs → Module B inputs → Module A inputs. Terraform cannot resolve this. Restructure: extract shared data into a third module or use data sources.

## Not Using `moved` Blocks During Refactoring
Renaming a resource or moving it into a module without a `moved` block causes Terraform to destroy the old resource and create a new one. Use `moved` blocks — see [code-patterns.md](code-patterns.md).

## Provisioners Instead of Proper Configuration
```hcl
# BAD — local-exec to run az CLI commands
provisioner "local-exec" {
  command = "az keyvault secret set --vault-name ${self.name} --name db-password --value ${var.password}"
}

# GOOD — native resource
resource "azurerm_key_vault_secret" "db_password" {
  name         = "db-password"
  value        = var.password
  key_vault_id = azurerm_key_vault.main.id
}
```
Provisioners are a last resort — they don't track state, can't be planned, and break idempotency. Use `azapi_resource` if azurerm lacks the resource.
