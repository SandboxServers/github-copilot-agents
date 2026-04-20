# Provider Landscape

## azurerm Provider
The primary provider for Azure Resource Manager resources. Covers ~95% of GA Azure resources.

### Configuration
```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"    # Pin to major version, allow minor/patch updates
    }
  }
}

provider "azurerm" {
  features {
    key_vault {
      purge_soft_delete_on_destroy    = false  # Safety: don't purge on destroy
      recover_soft_deleted_key_vaults = true
    }
    resource_group {
      prevent_deletion_if_contains_resources = true  # Safety: don't delete populated RGs
    }
    api_management {
      purge_soft_delete_on_destroy = false
    }
  }
  
  subscription_id = var.subscription_id  # Required in v4 (no longer inferred)
}
```

### Authentication Methods (in preference order)
```hcl
# 1. OIDC / Workload Identity Federation (CI/CD — best practice)
provider "azurerm" {
  features {}
  subscription_id     = var.subscription_id
  use_oidc            = true
  # Client ID, tenant ID from environment or variables
}

# 2. Managed Identity (Azure-hosted agents)
provider "azurerm" {
  features {}
  subscription_id     = var.subscription_id
  use_msi             = true
  # Optional: msi_endpoint, client_id for user-assigned MI
}

# 3. Service Principal + Certificate
provider "azurerm" {
  features {}
  subscription_id     = var.subscription_id
  client_id           = var.client_id
  client_certificate_path     = var.cert_path
  client_certificate_password = var.cert_password
  tenant_id           = var.tenant_id
}

# 4. Azure CLI (local development only)
provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
  # Automatically uses `az login` credentials
}
```

### Features Block Important Settings
```hcl
features {
  key_vault {
    purge_soft_delete_on_destroy    = false  # Don't purge soft-deleted KV on destroy
    recover_soft_deleted_key_vaults = true   # Recover if KV was soft-deleted
  }
  resource_group {
    prevent_deletion_if_contains_resources = true  # Prevent accidental RG deletion
  }
  log_analytics_workspace {
    permanently_delete_on_destroy = false  # Protect LAW data
  }
  virtual_machine {
    delete_os_disk_on_deletion     = true  # Clean up OS disk on VM destroy
    graceful_shutdown              = true   # Wait for graceful shutdown
    skip_shutdown_and_force_delete = false  # Don't force-delete
  }
}
```

### v3 → v4 Breaking Changes
Key changes when upgrading:
- `subscription_id` is now REQUIRED in provider block (no longer inferred from CLI)
- Many `azurerm_*` resources dropped — replaced by dedicated resource types
- Provider-level feature flags changed names/defaults
- Some resource attributes renamed or restructured
- Use `aztfmigrate` tool for automated migration

## azapi Provider
For Azure resources not yet in azurerm, preview APIs, or when you need to manage specific API versions.

```hcl
terraform {
  required_providers {
    azapi = {
      source  = "Azure/azapi"
      version = "~> 2.0"
    }
  }
}

# Manage a resource by its ARM resource type and API version
resource "azapi_resource" "container_app_environment" {
  type      = "Microsoft.App/managedEnvironments@2024-03-01"
  name      = "${local.name_prefix}-cae"
  location  = azurerm_resource_group.main.location
  parent_id = azurerm_resource_group.main.id

  body = {
    properties = {
      zoneRedundant           = true
      infrastructureSubnetId  = azurerm_subnet.container_apps.id
      workloadProfiles = [{
        name                = "consumption"
        workloadProfileType = "Consumption"
      }]
    }
  }

  tags = local.common_tags
}

# Update a property on an existing resource (PATCH)
resource "azapi_update_resource" "diagnostic_settings" {
  type        = "Microsoft.Insights/diagnosticSettings@2021-05-01-preview"
  resource_id = "${azurerm_key_vault.main.id}/providers/Microsoft.Insights/diagnosticSettings/audit"
  
  body = {
    properties = {
      workspaceId = azurerm_log_analytics_workspace.main.id
      logs = [{
        category = "AuditEvent"
        enabled  = true
      }]
    }
  }
}

# Read-only data source for any ARM resource
resource "azapi_resource_action" "list_keys" {
  type        = "Microsoft.Storage/storageAccounts@2023-05-01"
  resource_id = azurerm_storage_account.main.id
  action      = "listKeys"
  method      = "POST"
}
```

### When to Use azapi vs azurerm
| Use azapi When | Use azurerm When |
|---|---|
| Resource not in azurerm yet | Resource exists in azurerm |
| Need specific API version control | Standard API version is fine |
| Preview/experimental features | GA features |
| Day-0 support for new services | Stable, well-tested resources |
| Need `azapi_update_resource` for partial updates | Full resource management |

### aztfmigrate: azapi → azurerm Migration
```bash
# When azurerm adds support for a resource you're managing with azapi
aztfmigrate plan     # Show migration plan
aztfmigrate migrate  # Generate azurerm config + state migration commands
```

## azuread Provider
Separate provider for Entra ID (Azure AD) resources. Not part of azurerm.

```hcl
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 3.0"
    }
  }
}

# App registration
resource "azuread_application" "workload" {
  display_name = "${local.name_prefix}-app"
  owners       = [data.azuread_client_config.current.object_id]
}

# Service principal
resource "azuread_service_principal" "workload" {
  client_id = azuread_application.workload.client_id
  owners    = [data.azuread_client_config.current.object_id]
}

# Federated identity credential (OIDC for GitHub Actions)
resource "azuread_application_federated_identity_credential" "github" {
  application_id = azuread_application.workload.id
  display_name   = "github-actions-deploy"
  description    = "GitHub Actions OIDC for main branch deployments"
  audiences      = ["api://AzureADTokenExchange"]
  issuer         = "https://token.actions.githubusercontent.com"
  subject        = "repo:org/repo:ref:refs/heads/main"
}
```

## Provider Version Pinning Strategy
```hcl
# In required_providers — pin to major, allow minor/patch
azurerm = {
  source  = "hashicorp/azurerm"
  version = "~> 4.0"      # >= 4.0.0, < 5.0.0
}

# Lock file: .terraform.lock.hcl — commit this to source control
# It locks the exact version + hashes used
# Update with: terraform init -upgrade
```
