# Common Gotchas

## Replacement Triggers
These properties force resource destruction and recreation when changed. Plan carefully.

| Resource | Property | Impact |
|---|---|---|
| `azurerm_resource_group` | `name`, `location` | Destroys ALL contained resources |
| `azurerm_storage_account` | `name`, `account_kind`, `account_replication_type` (some changes) | Data loss risk |
| `azurerm_key_vault` | `name`, `location` | Goes to soft-delete state |
| `azurerm_linux_web_app` | `name`, `resource_group_name` | App URL changes |
| `azurerm_kubernetes_cluster` | `name`, `location`, `dns_prefix` | Full cluster rebuild |
| `azurerm_cosmosdb_account` | `name`, `location` | Data loss risk |
| `azurerm_sql_server` | `name`, `location` | All databases destroyed |
| `azurerm_virtual_network` | `name`, `location` | All subnets, peerings destroyed |

**Mitigation**: Use `lifecycle { prevent_destroy = true }` on critical resources.

## Azure-Mutated Properties (Need ignore_changes)
Azure modifies these properties outside Terraform. Without `ignore_changes`, every plan shows drift.

```hcl
# AKS: Autoscaler manages node count
resource "azurerm_kubernetes_cluster" "main" {
  # ...
  lifecycle {
    ignore_changes = [
      default_node_pool[0].node_count,
      # If using cluster autoscaler, node count changes constantly
    ]
  }
}

# Tags set by Azure Policy
resource "azurerm_resource_group" "main" {
  # ...
  lifecycle {
    ignore_changes = [
      tags["CreatedBy"],
      tags["LastModified"],
      tags["CreatedDate"],
    ]
  }
}

# App Service: Deployment-managed properties
resource "azurerm_linux_web_app" "main" {
  # ...
  lifecycle {
    ignore_changes = [
      site_config[0].application_stack,   # Managed by CI/CD deployment
      app_settings["WEBSITE_RUN_FROM_PACKAGE"],  # Set by deployment
    ]
  }
}

# Function App: Runtime-managed settings
resource "azurerm_linux_function_app" "main" {
  # ...
  lifecycle {
    ignore_changes = [
      app_settings["WEBSITE_CONTENTSHARE"],
      app_settings["WEBSITE_RUN_FROM_PACKAGE"],
    ]
  }
}
```

## Role Assignment Propagation Delay
Azure AD role assignments take 30-60 seconds to propagate. Terraform doesn't wait.

```hcl
# Problem: Key Vault access fails because role assignment hasn't propagated
resource "azurerm_role_assignment" "kv_secrets_officer" {
  scope                = azurerm_key_vault.main.id
  role_definition_name = "Key Vault Secrets Officer"
  principal_id         = azurerm_user_assigned_identity.main.principal_id
}

# Solution: Add a time_sleep between role assignment and usage
resource "time_sleep" "wait_for_rbac" {
  depends_on      = [azurerm_role_assignment.kv_secrets_officer]
  create_duration = "60s"
}

resource "azurerm_key_vault_secret" "main" {
  depends_on   = [time_sleep.wait_for_rbac]
  name         = "my-secret"
  value        = "secret-value"
  key_vault_id = azurerm_key_vault.main.id
}

# Also useful: skip_service_principal_aad_check
resource "azurerm_role_assignment" "fast" {
  scope                            = azurerm_key_vault.main.id
  role_definition_name             = "Key Vault Secrets Officer"
  principal_id                     = azurerm_user_assigned_identity.main.principal_id
  skip_service_principal_aad_check = true  # Skip validation, creates faster
}
```

## API Rate Limiting / Throttling
Azure ARM API has rate limits per subscription and provider. Large deployments can hit them.

**Symptoms**: 429 errors, intermittent failures, `RetryableError` in plan/apply.

**Mitigations**:
- Use `-parallelism=5` (default is 10) for large deployments
- Split large configurations into smaller state files
- Stagger deployments across subscriptions
- `terraform apply -parallelism=2` for particularly sensitive operations

## Dependency Issues

### Implicit vs Explicit Dependencies
```hcl
# GOOD: Implicit dependency — Terraform figures it out from references
resource "azurerm_subnet" "main" {
  virtual_network_name = azurerm_virtual_network.main.name  # Creates dependency
}

# AVOID unless necessary: Explicit dependency
resource "azurerm_subnet" "main" {
  depends_on = [azurerm_virtual_network.main]  # Only when implicit isn't possible
}

# When depends_on IS necessary:
# - Conditional resources (count/for_each) that other resources need
# - Policy assignments that must exist before resources
# - Custom Script Extensions that depend on network config
```

### create_before_destroy
```hcl
# Problem: DNS record points to old IP during replacement
resource "azurerm_public_ip" "main" {
  name                = "${local.name_prefix}-pip"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"

  lifecycle {
    create_before_destroy = true  # New IP created before old one deleted
  }
}
# CAVEAT: Only works if the name can be different (temporary names)
# Azure requires unique names — you may need to use random_id in the name
```

## Timeout Configuration
```hcl
resource "azurerm_kubernetes_cluster" "main" {
  # ... config ...

  timeouts {
    create = "90m"   # AKS can take 30-60 min to create
    update = "90m"
    delete = "60m"
    read   = "5m"
  }
}

# Resources that commonly need extended timeouts:
# - AKS clusters: 60-90 min create
# - Virtual Network Gateways: 45-60 min
# - Azure Firewall: 30-45 min
# - SQL Managed Instance: 4-6 hours
# - ExpressRoute circuits: 30-60 min
```

## Import Gotchas
```hcl
# Problem: Imported resource has properties not in your config
# Solution: Run plan after import, add missing properties to match

# Problem: Resource ID format varies by resource type
# AzureRM uses ARM resource IDs:
# /subscriptions/{sub}/resourceGroups/{rg}/providers/{provider}/{type}/{name}

# Problem: Import doesn't generate code
# Solution: Use `terraform plan` after import to see what properties need to be added
# Or use: terraform show -json | jq to extract the imported state
```

## Provider Upgrade Workflow
1. Read the provider CHANGELOG for breaking changes
2. Update version constraint in `required_providers`
3. Run `terraform init -upgrade`
4. Run `terraform plan` — expect some drift from renamed/restructured attributes
5. Fix code to match new provider expectations
6. Run `terraform plan` again — should show no unexpected changes
7. Apply in dev first, then staging, then prod
8. Commit updated `.terraform.lock.hcl`

## Naming Anti-Patterns
```hcl
# BAD: Hardcoded environment-specific names
resource "azurerm_resource_group" "main" {
  name = "rg-myapp-prod"    # Can't reuse for dev/staging
}

# GOOD: Parameterized naming
resource "azurerm_resource_group" "main" {
  name = "${var.project}-${var.environment}-rg"
}

# BAD: Random suffixes without purpose
resource "azurerm_storage_account" "main" {
  name = "st${random_string.suffix.result}"  # Unreadable, can't predict
}

# GOOD: Structured naming with uniqueness
resource "azurerm_storage_account" "main" {
  name = "st${var.project}${var.environment}${var.location_short}"
}
```
