# Managed Identity & RBAC Patterns

## System-Assigned Identity
```hcl
resource "azurerm_linux_web_app" "main" {
  name                = "${local.name_prefix}-app"
  # ... other config ...
  
  identity {
    type = "SystemAssigned"
  }
}

# Role assignment using the system-assigned identity
resource "azurerm_role_assignment" "app_to_kv" {
  scope                = azurerm_key_vault.main.id
  role_definition_name = "Key Vault Secrets User"
  principal_id         = azurerm_linux_web_app.main.identity[0].principal_id
}
```

## User-Assigned Identity (Preferred for Terraform)
User-assigned is preferred because:
- Identity lifecycle is independent from the resource
- Can be created first, then assigned — avoids chicken-and-egg with role assignments
- One identity can be shared across multiple resources
- No identity recreation when resource is replaced

```hcl
resource "azurerm_user_assigned_identity" "workload" {
  name                = "${local.name_prefix}-id"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  tags                = local.common_tags
}

# Assign roles BEFORE attaching to resources
resource "azurerm_role_assignment" "workload_kv" {
  scope                            = azurerm_key_vault.main.id
  role_definition_name             = "Key Vault Secrets User"
  principal_id                     = azurerm_user_assigned_identity.workload.principal_id
  skip_service_principal_aad_check = true
}

resource "time_sleep" "wait_for_rbac" {
  depends_on      = [azurerm_role_assignment.workload_kv]
  create_duration = "60s"
}

# Attach to resource after roles propagate
resource "azurerm_linux_web_app" "main" {
  depends_on = [time_sleep.wait_for_rbac]
  # ... config ...
  
  identity {
    type         = "UserAssigned"
    identity_ids = [azurerm_user_assigned_identity.workload.id]
  }
}
```

## Combined Identity Types
```hcl
resource "azurerm_kubernetes_cluster" "main" {
  # ... config ...
  
  identity {
    type         = "SystemAssigned, UserAssigned"
    identity_ids = [azurerm_user_assigned_identity.aks.id]
  }
  # SystemAssigned gives AKS its own identity for control plane operations
  # UserAssigned gives pods access via workload identity
}
```

## Federated Identity Credentials (OIDC for CI/CD)
```hcl
# App registration for CI/CD
resource "azuread_application" "cicd" {
  display_name = "${local.name_prefix}-cicd"
  owners       = [data.azuread_client_config.current.object_id]
}

resource "azuread_service_principal" "cicd" {
  client_id = azuread_application.cicd.client_id
  owners    = [data.azuread_client_config.current.object_id]
}

# GitHub Actions OIDC federation
resource "azuread_application_federated_identity_credential" "github_main" {
  application_id = azuread_application.cicd.id
  display_name   = "github-actions-main"
  description    = "GitHub Actions deployments from main branch"
  audiences      = ["api://AzureADTokenExchange"]
  issuer         = "https://token.actions.githubusercontent.com"
  subject        = "repo:${var.github_org}/${var.github_repo}:ref:refs/heads/main"
}

# Azure Pipelines OIDC federation
resource "azuread_application_federated_identity_credential" "ado_prod" {
  application_id = azuread_application.cicd.id
  display_name   = "ado-prod-deploy"
  description    = "Azure Pipelines production deployment"
  audiences      = ["api://AzureADTokenExchange"]
  issuer         = "https://vstoken.dev.azure.com/${var.ado_org_id}"
  subject        = "sc://${var.ado_org}/${var.ado_project}/${var.service_connection_name}"
}

# Grant the SP access to deploy
resource "azurerm_role_assignment" "cicd_contributor" {
  scope                = data.azurerm_subscription.current.id
  role_definition_name = "Contributor"
  principal_id         = azuread_service_principal.cicd.object_id
}
```

## Custom Role Definitions
```hcl
resource "azurerm_role_definition" "terraform_deployer" {
  name        = "Terraform Deployer"
  scope       = data.azurerm_subscription.current.id
  description = "Custom role for Terraform CI/CD with least-privilege access."

  permissions {
    actions = [
      "Microsoft.Resources/subscriptions/resourceGroups/*",
      "Microsoft.Web/*",
      "Microsoft.KeyVault/*",
      "Microsoft.Network/*",
      "Microsoft.Storage/*",
    ]
    not_actions = [
      "Microsoft.Authorization/roleAssignments/delete",
    ]
    data_actions = [
      "Microsoft.KeyVault/vaults/secrets/readMetadata/action",
    ]
  }

  assignable_scopes = [
    data.azurerm_subscription.current.id,
  ]
}
```

## Data Plane RBAC Patterns

### Key Vault (RBAC Mode)
```hcl
resource "azurerm_key_vault" "main" {
  # ... config ...
  enable_rbac_authorization = true  # Use RBAC, not access policies
}

# Different roles for different access levels
resource "azurerm_role_assignment" "app_secrets_user" {
  scope                = azurerm_key_vault.main.id
  role_definition_name = "Key Vault Secrets User"      # Read secrets
  principal_id         = azurerm_user_assigned_identity.app.principal_id
}

resource "azurerm_role_assignment" "admin_secrets_officer" {
  scope                = azurerm_key_vault.main.id
  role_definition_name = "Key Vault Secrets Officer"    # Read + write secrets
  principal_id         = var.admin_group_object_id
}
```

### Storage Account
```hcl
resource "azurerm_role_assignment" "app_blob_reader" {
  scope                = azurerm_storage_account.main.id
  role_definition_name = "Storage Blob Data Reader"
  principal_id         = azurerm_user_assigned_identity.app.principal_id
}

# Scoped to a specific container
resource "azurerm_role_assignment" "app_container_contributor" {
  scope                = "${azurerm_storage_account.main.id}/blobServices/default/containers/${azurerm_storage_container.uploads.name}"
  role_definition_name = "Storage Blob Data Contributor"
  principal_id         = azurerm_user_assigned_identity.app.principal_id
}
```

### Cosmos DB
```hcl
resource "azurerm_cosmosdb_sql_role_assignment" "app_contributor" {
  resource_group_name = azurerm_resource_group.main.name
  account_name        = azurerm_cosmosdb_account.main.name
  role_definition_id  = "${azurerm_cosmosdb_account.main.id}/sqlRoleDefinitions/00000000-0000-0000-0000-000000000002"  # Built-in Contributor
  principal_id        = azurerm_user_assigned_identity.app.principal_id
  scope               = azurerm_cosmosdb_account.main.id
}
```

## Multi-Resource Identity Pattern
```hcl
# One identity, multiple role assignments, multiple consumers
resource "azurerm_user_assigned_identity" "shared_workload" {
  name                = "${local.name_prefix}-shared-id"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
}

# Role assignments for each service the workload needs
locals {
  role_assignments = {
    kv_reader     = { scope = azurerm_key_vault.main.id, role = "Key Vault Secrets User" }
    blob_writer   = { scope = azurerm_storage_account.main.id, role = "Storage Blob Data Contributor" }
    sql_reader    = { scope = azurerm_mssql_server.main.id, role = "SQL DB Contributor" }
  }
}

resource "azurerm_role_assignment" "workload" {
  for_each = local.role_assignments

  scope                            = each.value.scope
  role_definition_name             = each.value.role
  principal_id                     = azurerm_user_assigned_identity.shared_workload.principal_id
  skip_service_principal_aad_check = true
}
```
