# Code Patterns

## for_each (Preferred Over count)
```hcl
# for_each with a map — resources keyed by name, not index
variable "subnets" {
  type = map(object({
    address_prefixes  = list(string)
    service_endpoints = optional(list(string), [])
  }))
}

resource "azurerm_subnet" "this" {
  for_each = var.subnets

  name                 = each.key
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = each.value.address_prefixes
  service_endpoints    = each.value.service_endpoints
}

# Why for_each > count:
# - Adding/removing items doesn't reindex everything
# - Resources are keyed by name: azurerm_subnet.this["app-gateway"]
# - Easier to reference: module.network.subnet_ids["app-gateway"]
```

## count (When to Use)
```hcl
# count is appropriate for binary conditionals (0 or 1)
resource "azurerm_private_dns_zone" "this" {
  count = var.enable_private_dns ? 1 : 0
  name                = "privatelink.azurewebsites.net"
  resource_group_name = azurerm_resource_group.main.name
}

# Reference with [0] when count is used
output "dns_zone_id" {
  value = var.enable_private_dns ? azurerm_private_dns_zone.this[0].id : null
}
```

## for Expressions
```hcl
locals {
  # Transform a list into a map
  subnet_ids = { for s in azurerm_subnet.this : s.name => s.id }
  
  # Filter a collection
  production_apps = { for k, v in var.apps : k => v if v.environment == "prod" }
  
  # Flatten nested structures
  nsg_rules = flatten([
    for subnet_name, subnet in var.subnets : [
      for rule_name, rule in subnet.nsg_rules : {
        subnet_name = subnet_name
        rule_name   = rule_name
        priority    = rule.priority
        direction   = rule.direction
        access      = rule.access
        protocol    = rule.protocol
        source      = rule.source
        destination = rule.destination
      }
    ]
  ])
  
  # Create a map from flattened list (for for_each)
  nsg_rules_map = { for rule in local.nsg_rules : "${rule.subnet_name}-${rule.rule_name}" => rule }
}
```

## Variable Validation
```hcl
variable "environment" {
  description = "Deployment environment."
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "vnet_cidr" {
  description = "CIDR block for the virtual network."
  type        = string
  validation {
    condition     = can(cidrhost(var.vnet_cidr, 0))
    error_message = "Must be a valid CIDR block (e.g., 10.0.0.0/16)."
  }
}

variable "name" {
  description = "Resource name. Must be 3-24 characters, lowercase alphanumeric."
  type        = string
  validation {
    condition     = can(regex("^[a-z0-9]{3,24}$", var.name))
    error_message = "Name must be 3-24 lowercase alphanumeric characters."
  }
}

# Multiple validations on one variable (TF 1.9+: each can have a custom error)
variable "sku" {
  description = "SKU for the resource."
  type        = string
  validation {
    condition     = contains(["Basic", "Standard", "Premium"], var.sku)
    error_message = "SKU must be Basic, Standard, or Premium."
  }
  validation {
    condition     = var.sku != "Basic" || !var.enable_private_endpoints
    error_message = "Basic SKU does not support private endpoints."
  }
}
```

## optional() Object Attributes
```hcl
variable "diagnostics" {
  description = "Diagnostic settings configuration."
  type = object({
    log_analytics_workspace_id = string
    storage_account_id         = optional(string)        # null if not provided
    retention_days             = optional(number, 30)     # defaults to 30
    log_categories             = optional(list(string), ["AuditEvent", "AllMetrics"])
  })
}
```

## Lifecycle Rules
```hcl
resource "azurerm_key_vault" "main" {
  # ... config ...
  
  lifecycle {
    prevent_destroy = true    # Protect critical resources from accidental destruction
  }
}

resource "azurerm_kubernetes_cluster" "main" {
  # ... config ...
  
  lifecycle {
    ignore_changes = [
      default_node_pool[0].node_count,    # Managed by autoscaler
      tags["LastModified"],                # Set by Azure Policy
    ]
  }
}

resource "azurerm_app_service_certificate" "main" {
  # ... config ...
  
  lifecycle {
    create_before_destroy = true    # Zero-downtime cert rotation
  }
}
```

## Dynamic Blocks
```hcl
resource "azurerm_network_security_group" "this" {
  name                = "${local.name_prefix}-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  dynamic "security_rule" {
    for_each = var.nsg_rules
    content {
      name                       = security_rule.value.name
      priority                   = security_rule.value.priority
      direction                  = security_rule.value.direction
      access                     = security_rule.value.access
      protocol                   = security_rule.value.protocol
      source_port_range          = security_rule.value.source_port_range
      destination_port_range     = security_rule.value.destination_port_range
      source_address_prefix      = security_rule.value.source_address_prefix
      destination_address_prefix = security_rule.value.destination_address_prefix
    }
  }

  tags = local.common_tags
}
```

## Preconditions and Postconditions (TF 1.2+)
```hcl
resource "azurerm_linux_web_app" "main" {
  # ... config ...
  
  lifecycle {
    precondition {
      condition     = data.azurerm_service_plan.main.sku_name != "F1"
      error_message = "Cannot deploy to a Free tier App Service Plan."
    }
    
    postcondition {
      condition     = self.default_hostname != ""
      error_message = "App Service did not receive a default hostname."
    }
  }
}

data "azurerm_subscription" "current" {}

# Validate assumptions about the deployment context
resource "terraform_data" "validate_subscription" {
  lifecycle {
    precondition {
      condition     = data.azurerm_subscription.current.display_name != "Visual Studio Enterprise"
      error_message = "Do not deploy to the Visual Studio Enterprise subscription."
    }
  }
}
```

## Check Blocks (TF 1.5+ — Continuous Validation)
```hcl
# Check blocks run on every plan/apply but DON'T block deployments
# They produce warnings, not errors — good for drift detection
check "certificate_expiry" {
  data "azurerm_key_vault_certificate" "app" {
    name         = "app-cert"
    key_vault_id = azurerm_key_vault.main.id
  }

  assert {
    condition     = timecmp(plantimestamp(), timeadd(data.azurerm_key_vault_certificate.app.expires, "-720h")) < 0
    error_message = "Certificate expires within 30 days — renew immediately."
  }
}
```

## Key HCL Functions
```hcl
locals {
  # try() — return first successful expression, useful for optional nested attrs
  vm_size = try(var.override_config.vm_size, "Standard_D2s_v5")
  
  # coalesce() — first non-null, non-empty value
  location = coalesce(var.location, data.azurerm_resource_group.main.location)
  
  # one() — extract single element from a set/list (errors if >1)
  subnet_id = one(azurerm_subnet.this[*].id)
  
  # cidrsubnet() — calculate subnet CIDRs from a VNet CIDR
  subnets = {
    "app-gateway"       = cidrsubnet(var.vnet_cidr, 8, 0)    # /24 from /16
    "workload"          = cidrsubnet(var.vnet_cidr, 8, 1)
    "private-endpoints" = cidrsubnet(var.vnet_cidr, 8, 2)
  }
  
  # merge() — combine maps (last wins on conflicts)
  tags = merge(local.common_tags, var.extra_tags)
  
  # flatten() — flatten nested lists
  all_subnet_ids = flatten([for k, v in module.networks : values(v.subnet_ids)])
}
```

## Moved Blocks (Refactoring)
```hcl
# Rename a resource without destroy/recreate
moved {
  from = azurerm_resource_group.rg
  to   = azurerm_resource_group.main
}

# Move a resource into a module
moved {
  from = azurerm_virtual_network.hub
  to   = module.hub_network.azurerm_virtual_network.main
}

# Move between module instances (for_each key change)
moved {
  from = module.app["old-name"]
  to   = module.app["new-name"]
}

# After successful apply, remove moved blocks — they're transitional
```

## Data Source Patterns
```hcl
# Current client identity
data "azurerm_client_config" "current" {}
data "azurerm_subscription" "current" {}

# Look up existing resources
data "azurerm_resource_group" "existing" {
  name = var.existing_rg_name
}

# Look up by tags (when name isn't known)
data "azurerm_resources" "tagged" {
  type = "Microsoft.KeyVault/vaults"
  required_tags = {
    Environment = var.environment
    Project     = var.project
  }
}
```
