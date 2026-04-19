## Testing Patterns

### Terraform Test (Native — `.tftest.hcl`)
```hcl
# tests/unit/main.tftest.hcl
run "validate_resource_group_name" {
  command = plan

  variables {
    name        = "myapp"
    environment = "dev"
    location    = "eastus2"
  }

  assert {
    condition     = azurerm_resource_group.main.name == "rg-myapp-dev"
    error_message = "Resource group name doesn't match expected pattern."
  }
}

run "validate_tags" {
  command = plan

  variables {
    name        = "myapp"
    environment = "dev"
  }

  assert {
    condition     = azurerm_resource_group.main.tags["Environment"] == "dev"
    error_message = "Environment tag not set correctly."
  }
}
```

### Test Tooling Stack (AVM Standard)
| Tool | Purpose | What It Catches |
|------|---------|----------------|
| `terraform validate` | Syntax and internal consistency | Typos, missing required arguments, type mismatches |
| `terraform fmt -check` | Code formatting | Inconsistent indentation, spacing |
| `tflint` (with azurerm ruleset) | Linting, AVM compliance | Invalid values, deprecated features, best practice violations |
| `checkov` | Security scanning | Insecure defaults, missing encryption, public access |
| `terraform test` | Unit and integration testing | Logic errors, incorrect outputs, broken conditions |
| `terraform plan` + review | Plan validation | Unexpected changes, resource replacement, drift |
| `conftest` + OPA | Policy compliance (WAF) | WAF violations, organizational policy breaches |
| `terrafmt` | HCL formatting in docs | Malformed HCL in markdown documentation |

### Pre-commit Workflow
```
1. terraform fmt -check -recursive
2. terraform validate
3. tflint --init && tflint
4. checkov -d . --framework terraform
5. terraform test (if .tftest.hcl files exist)
6. terraform plan (in CI — catch surprises before apply)
```
