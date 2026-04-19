## State Management

### Backend Configuration

| Backend | Best For | Key Considerations |
|---------|----------|-------------------|
| **azurerm (Azure Storage)** | Most Azure teams | Lock via blob lease, encrypt at rest, use managed identity auth |
| **remote (Terraform Cloud/Enterprise)** | Teams using TFC/TFE | Built-in locking, run history, policy-as-code |
| **s3 + DynamoDB** | Multi-cloud teams | If already using AWS for state |
| **local** | Dev/testing only | Never for shared/production |

### Azure Storage Backend Best Practices
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "stterraformstate"
    container_name       = "tfstate"
    key                  = "project/environment.tfstate"
    use_azuread_auth     = true  # Managed identity, no storage keys
  }
}
```
- **One state file per deployment scope** — don't put dev/staging/prod in one state
- **Enable versioning and soft delete** on state storage account
- **Lock the state storage account** — delete lock, no public access, private endpoint if possible
- **Never store secrets in state** — but know that they end up there anyway (use sensitive outputs, encrypt backend)

### Workspace Strategies

| Strategy | Structure | Best For |
|----------|-----------|---------|
| **Directory-per-environment** | `envs/dev/`, `envs/staging/`, `envs/prod/` | Clear separation, different backends per env |
| **Workspace-per-environment** | Single config, `terraform workspace` | Simple stacks, same structure across envs |
| **Terragrunt** | DRY config with `terragrunt.hcl` | Complex multi-account, multi-region |

### State Operations to Know
- `terraform state mv` — rename/restructure without recreating resources
- `terraform state rm` — remove resource from state without destroying it (adopt existing resources)
- `terraform import` — bring existing Azure resources into state
- `terraform state pull/push` — manual state manipulation (emergency only)
- **Always backup state before state operations** — `terraform state pull > backup.tfstate`
