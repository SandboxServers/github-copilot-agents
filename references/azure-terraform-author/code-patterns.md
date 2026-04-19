## Code Patterns

### Variable Validation Blocks
```hcl
variable "environment" {
  type        = string
  description = "Deployment environment"
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "cidr_range" {
  type        = string
  description = "VNet CIDR range"
  validation {
    condition     = can(cidrhost(var.cidr_range, 0))
    error_message = "Must be a valid CIDR notation."
  }
}

variable "name" {
  type        = string
  description = "Resource name prefix"
  validation {
    condition     = length(var.name) >= 3 && length(var.name) <= 24
    error_message = "Name must be 3-24 characters."
  }
}
```

### Lifecycle Rules
```hcl
resource "azurerm_key_vault" "main" {
  # ... configuration ...

  lifecycle {
    prevent_destroy = true  # Stateful resource — don't accidentally delete
  }
}

resource "azurerm_kubernetes_cluster" "main" {
  # ... configuration ...

  lifecycle {
    ignore_changes = [
      default_node_pool[0].node_count,  # Managed by cluster autoscaler
      tags["CreatedDate"],               # Set by Azure Policy
    ]
  }
}
```

### Conditional Resource Creation
```hcl
variable "enable_monitoring" {
  type    = bool
  default = true
}

resource "azurerm_log_analytics_workspace" "main" {
  count = var.enable_monitoring ? 1 : 0
  # ...
}

# Reference with conditional
resource "azurerm_monitor_diagnostic_setting" "main" {
  count                      = var.enable_monitoring ? 1 : 0
  log_analytics_workspace_id = azurerm_log_analytics_workspace.main[0].id
  # ...
}
```

### Dynamic Blocks
```hcl
variable "nsg_rules" {
  type = list(object({
    name                       = string
    priority                   = number
    direction                  = string
    access                     = string
    protocol                   = string
    source_port_range          = string
    destination_port_range     = string
    source_address_prefix      = string
    destination_address_prefix = string
  }))
}

resource "azurerm_network_security_group" "main" {
  name                = "nsg-${var.name}"
  location            = var.location
  resource_group_name = var.resource_group_name

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
}
```

### Moved Blocks (Refactoring Without Destroy)
```hcl
# When renaming a resource or moving into a module
moved {
  from = azurerm_resource_group.main
  to   = module.core.azurerm_resource_group.main
}
```
