## Managed Identity Patterns

### System-Assigned
```hcl
resource "azurerm_linux_web_app" "main" {
  # ...
  identity {
    type = "SystemAssigned"
  }
}

resource "azurerm_role_assignment" "app_to_storage" {
  scope                = azurerm_storage_account.main.id
  role_definition_name = "Storage Blob Data Reader"
  principal_id         = azurerm_linux_web_app.main.identity[0].principal_id
}
```

### User-Assigned (Preferred for Terraform — Survives Recreate)
```hcl
resource "azurerm_user_assigned_identity" "app" {
  name                = "id-${var.name}-${var.environment}"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
}

resource "azurerm_role_assignment" "app_to_keyvault" {
  scope                = azurerm_key_vault.main.id
  role_definition_name = "Key Vault Secrets User"
  principal_id         = azurerm_user_assigned_identity.app.principal_id
}

resource "azurerm_linux_web_app" "main" {
  # ...
  identity {
    type         = "UserAssigned"
    identity_ids = [azurerm_user_assigned_identity.app.id]
  }
}
```

**Why user-assigned is better for Terraform**: system-assigned identity is destroyed when the resource is recreated → all role assignments break → downstream resources lose access. User-assigned identity survives resource recreation.
