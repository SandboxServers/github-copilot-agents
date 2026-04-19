## State and Deployment Model

### Bicep vs Terraform State
| Aspect | Bicep | Terraform |
|---|---|---|
| **State** | Azure is the state store — no separate state file | `.tfstate` file (local or remote backend) |
| **Drift detection** | What-if compares template to live Azure | `terraform plan` compares state to config |
| **Idempotency** | ARM handles — same template = same result | Terraform provider handles |
| **Deployment history** | Azure keeps deployment history per scope | State file tracks known resources |
| **Import** | Not applicable — resources already in Azure | `terraform import` adds to state |
| **Locking** | ARM handles concurrent deployment locking | State file locking (backend-dependent) |

### Deployment Modes
- **Incremental** (default) — add/update resources in template, leave others alone
- **Complete** — add/update resources in template, **delete** resources not in template (JSON templates only, not recommended for Bicep)

### Deployment History
- Azure keeps last 800 deployments per scope
- Auto-purge kicks in when approaching limit
- Use `az deployment group list` to inspect history
