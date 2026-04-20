# State Management

## Azure Storage Backend Configuration

### Full Configuration
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "stterraformstate001"
    container_name       = "tfstate"
    key                  = "project/environment/terraform.tfstate"
    use_azuread_auth     = true    # RBAC instead of storage keys
    use_oidc             = true    # For CI/CD with workload identity
  }
}
```

### Partial Configuration (CI/CD Pattern)
```hcl
# In code — minimal backend block
terraform {
  backend "azurerm" {}
}

# At init time — pass config via CLI or file
# terraform init -backend-config=backend.hcl
# OR
# terraform init \
#   -backend-config="resource_group_name=rg-terraform-state" \
#   -backend-config="storage_account_name=stterraformstate001" \
#   -backend-config="container_name=tfstate" \
#   -backend-config="key=project/prod/terraform.tfstate" \
#   -backend-config="use_azuread_auth=true" \
#   -backend-config="use_oidc=true"
```

### Storage Account Hardening
```hcl
# The state storage account itself (managed separately or bootstrapped)
resource "azurerm_storage_account" "tfstate" {
  name                          = "stterraformstate001"
  resource_group_name           = azurerm_resource_group.tfstate.name
  location                      = azurerm_resource_group.tfstate.location
  account_tier                  = "Standard"
  account_replication_type      = "GRS"          # Geo-redundant for state
  min_tls_version               = "TLS1_2"
  shared_access_key_enabled     = false          # Force RBAC auth
  public_network_access_enabled = false          # Private endpoint only
  
  blob_properties {
    versioning_enabled = true                    # State file versioning
    delete_retention_policy {
      days = 30                                  # Soft delete for 30 days
    }
    container_delete_retention_policy {
      days = 30
    }
  }
}

# RBAC for CI/CD identity
resource "azurerm_role_assignment" "tfstate_contributor" {
  scope                = azurerm_storage_account.tfstate.id
  role_definition_name = "Storage Blob Data Contributor"
  principal_id         = azuread_service_principal.cicd.object_id
}
```

## State File Per Environment Strategy
```
stterraformstate001/
└── tfstate/
    ├── shared/core/terraform.tfstate        # Shared infra (DNS, hub network)
    ├── project-alpha/dev/terraform.tfstate
    ├── project-alpha/staging/terraform.tfstate
    ├── project-alpha/prod/terraform.tfstate
    └── project-beta/prod/terraform.tfstate
```

- One state file per deployment scope (environment + project)
- Separate state for shared/core infrastructure
- NEVER put dev and prod in the same state file

## Workspace Strategy

| Strategy | When to Use | How |
|---|---|---|
| Separate directories + state keys | Multiple environments | `backend.key = "project/env/terraform.tfstate"` |
| Terraform workspaces | Same config, different instances | `terraform workspace select prod` |
| Terragrunt | Complex multi-environment orchestration | DRY configuration with inheritance |

**Recommendation**: Separate directories with variable files (e.g., `envs/dev.tfvars`, `envs/prod.tfvars`) and different state keys. Workspaces are fine for simple cases but don't scale well when environments diverge.

## State Operations

### Import (Terraform 1.5+ import blocks — preferred)
```hcl
# Declarative import — lives in code, runs with plan/apply
import {
  to = azurerm_resource_group.main
  id = "/subscriptions/xxx/resourceGroups/my-rg"
}

# After successful import, remove the import block
# The resource now lives in state and code
```

```bash
# CLI import (legacy — still works)
terraform import azurerm_resource_group.main /subscriptions/xxx/resourceGroups/my-rg
```

### Moved Blocks (Refactoring Without Destroy)
```hcl
# Rename a resource
moved {
  from = azurerm_virtual_network.vnet
  to   = azurerm_virtual_network.hub
}

# Move into a module
moved {
  from = azurerm_virtual_network.hub
  to   = module.network.azurerm_virtual_network.main
}

# Move within a for_each
moved {
  from = azurerm_subnet.subnets["old-name"]
  to   = azurerm_subnet.subnets["new-name"]
}
```

### State Removal
```bash
# Remove from state without destroying the resource
terraform state rm 'azurerm_resource_group.old'

# List resources in state
terraform state list

# Show a specific resource in state
terraform state show 'azurerm_virtual_network.main'
```

### Force Unlock (Emergency Only)
```bash
# When state lock is stuck (e.g., pipeline crashed mid-apply)
terraform force-unlock <LOCK_ID>
# The lock ID is shown in the error message
# ONLY use when you're certain no other operation is running
```

## Cross-Stack Data Sharing

### Option 1: terraform_remote_state (Simple but Creates Coupling)
```hcl
data "terraform_remote_state" "network" {
  backend = "azurerm"
  config = {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "stterraformstate001"
    container_name       = "tfstate"
    key                  = "shared/network/terraform.tfstate"
    use_azuread_auth     = true
  }
}

# Reference outputs from the network state
resource "azurerm_private_endpoint" "kv" {
  subnet_id = data.terraform_remote_state.network.outputs.subnet_ids["private-endpoints"]
}
```

### Option 2: Data Sources (Preferred — No State Coupling)
```hcl
# Query existing infrastructure by name/tags
data "azurerm_virtual_network" "hub" {
  name                = "hub-vnet"
  resource_group_name = "rg-network-hub"
}

data "azurerm_subnet" "private_endpoints" {
  name                 = "snet-private-endpoints"
  virtual_network_name = data.azurerm_virtual_network.hub.name
  resource_group_name  = data.azurerm_virtual_network.hub.resource_group_name
}
```

### Option 3: Outputs File / Variable Injection
- Network stack outputs subnet IDs to a JSON file or pipeline variable
- Downstream stacks consume via `var.subnet_ids`
- Most flexible, least coupled, but requires orchestration layer

**Recommendation**: Use data sources for stable, well-known resources. Use `terraform_remote_state` only within the same team's stack. Use variable injection for cross-team dependencies.

## State Splitting Guidance
Split state when:
- State file > 1000 resources (plan/apply gets slow)
- Different lifecycle cadences (network changes rarely, app config changes often)
- Different teams own different layers
- Security boundaries (network team shouldn't have app secrets in their state)

Strategy:
1. Create new root module with the resources to split out
2. Use `moved` blocks or `terraform state mv` to migrate resources
3. Use `terraform state rm` to remove from old state
4. Validate with `terraform plan` on both stacks (should show no changes)

## Drift Detection
```bash
# Detect configuration drift
terraform plan -detailed-exitcode
# Exit code 0 = no changes, 1 = error, 2 = changes detected

# In CI/CD — run plan on a schedule to detect drift
# Alert if exit code is 2
```
