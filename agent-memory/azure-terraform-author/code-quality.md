# Code Quality

## Anti-Patterns and Corrections

### Using count when for_each is better
```hcl
# BAD: count with index — reordering the list destroys/recreates resources
variable "storage_accounts" {
  default = ["logs", "data", "backup"]
}

resource "azurerm_storage_account" "bad" {
  count = length(var.storage_accounts)
  name  = "st${var.storage_accounts[count.index]}${var.environment}"
  # If "data" is removed, "backup" shifts from index 2 to index 1 → destroyed and recreated
}

# GOOD: for_each with a set — removing an item only affects that item
resource "azurerm_storage_account" "good" {
  for_each = toset(var.storage_accounts)
  name     = "st${each.value}${var.environment}"
}
```

### Hardcoded values that should be variables
```hcl
# BAD
resource "azurerm_resource_group" "main" {
  name     = "rg-myapp-prod"
  location = "eastus2"
}

# GOOD
resource "azurerm_resource_group" "main" {
  name     = "${var.project}-${var.environment}-rg"
  location = var.location
}
```

### Missing variable validation
```hcl
# BAD: No validation — silent failures
variable "environment" {
  type = string
}

# GOOD: Fail fast with clear error
variable "environment" {
  type = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "cidr_block" {
  type = string
  validation {
    condition     = can(cidrhost(var.cidr_block, 0))
    error_message = "Must be a valid CIDR block (e.g., 10.0.0.0/16)."
  }
}
```

### Overly broad role assignments
```hcl
# BAD: Owner on subscription
resource "azurerm_role_assignment" "bad" {
  scope                = data.azurerm_subscription.current.id
  role_definition_name = "Owner"
  principal_id         = azurerm_user_assigned_identity.app.principal_id
}

# GOOD: Least privilege, scoped to resource
resource "azurerm_role_assignment" "good" {
  scope                = azurerm_key_vault.main.id
  role_definition_name = "Key Vault Secrets User"
  principal_id         = azurerm_user_assigned_identity.app.principal_id
}
```

### Magic numbers
```hcl
# BAD
resource "azurerm_kubernetes_cluster" "main" {
  default_node_pool {
    vm_size    = "Standard_D4s_v3"
    node_count = 3
    max_pods   = 110
  }
}

# GOOD
resource "azurerm_kubernetes_cluster" "main" {
  default_node_pool {
    vm_size    = var.system_node_pool_vm_size
    node_count = var.system_node_pool_count
    max_pods   = var.system_node_pool_max_pods
  }
}
```

### Sensitive values in state
```hcl
# BAD: Password visible in state and logs
variable "db_password" {
  type = string
}

# GOOD: Mark sensitive — hidden from plan output
variable "db_password" {
  type      = string
  sensitive = true
}

output "connection_string" {
  value     = "Server=${azurerm_mssql_server.main.fqdn};Password=${var.db_password}"
  sensitive = true
}
```

## PR Review Checklist

### Must-Have
- [ ] `terraform fmt` passes (no formatting drift)
- [ ] `terraform validate` passes
- [ ] `terraform plan` shows only expected changes
- [ ] No secrets or credentials in `.tf` files or `.tfvars` committed to repo
- [ ] Sensitive variables marked with `sensitive = true`
- [ ] All variables have `type`, `description`, and `validation` (where applicable)
- [ ] All outputs have `description`
- [ ] Resources that must not be destroyed have `prevent_destroy = true`
- [ ] Role assignments use least privilege
- [ ] `.terraform.lock.hcl` is committed and up to date

### Should-Have
- [ ] `tflint` passes with zero warnings
- [ ] `checkov` passes (or failures are documented and exempted)
- [ ] Integration tests pass in CI
- [ ] README is updated (via `terraform-docs`)
- [ ] Resources use `tags = local.common_tags` (or merge with resource-specific tags)
- [ ] Timeout blocks set on resources that commonly exceed defaults (AKS, VNet Gateway, SQL MI)
- [ ] `ignore_changes` set for Azure-mutated properties (node counts, deployment-managed settings)
- [ ] No `depends_on` that could be replaced with implicit reference dependencies

### Red Flags (Block the PR)
- [ ] `terraform plan` shows unexpected destroys
- [ ] `lifecycle { prevent_destroy = true }` removed from production databases, Key Vaults, storage accounts
- [ ] `Owner` or `Contributor` role assigned at subscription scope without documented justification
- [ ] Secrets stored in `terraform.tfvars` or hardcoded in `.tf` files
- [ ] State file stored locally or in a non-encrypted backend
- [ ] Provider version unpinned (`version = ">= 3.0"` with no upper bound)

## Security Scanning Integration

### tfsec (Aqua/Trivy)
```bash
# Run tfsec
tfsec . --format json --out tfsec-results.json

# Inline suppression (use sparingly, with documented reason)
# In the .tf file:
resource "azurerm_storage_account" "main" {
  #tfsec:ignore:azure-storage-default-action-deny  -- Public access required for static site
  # ...
}
```

### Checkov
```bash
# Run checkov
checkov -d . --output json > checkov-results.json

# Skip specific checks
checkov -d . --skip-check CKV_AZURE_35,CKV_AZURE_33
```

### Policy-as-Code (OPA/Conftest)
```rego
# policy/terraform.rego
package terraform

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "azurerm_storage_account"
  not resource.change.after.min_tls_version == "TLS1_2"
  msg := sprintf("Storage account %s must use TLS 1.2", [resource.address])
}

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "azurerm_storage_account"
  resource.change.after.allow_nested_items_to_be_public == true
  msg := sprintf("Storage account %s must not allow public blob access", [resource.address])
}
```

```bash
# Run against plan JSON
terraform show -json tfplan > plan.json
conftest test plan.json -p policy/
```
