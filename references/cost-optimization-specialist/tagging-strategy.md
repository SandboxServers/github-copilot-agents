## Tagging Strategy for Cost Allocation

### Minimum Viable Tagging

| Tag | Purpose | Example |
|---|---|---|
| `CostCenter` | Finance allocation | `CC-12345` |
| `Environment` | Dev/test/staging/prod identification | `Production` |
| `Owner` | Accountability | `team-platform@contoso.com` |
| `Project` | Project-level tracking | `ProjectMeridian` |
| `Application` | Application grouping | `DhadgarConsole` |

### Advanced Tags

| Tag | Purpose | Example |
|---|---|---|
| `Department` | Org-level rollup | `Engineering` |
| `DataClassification` | Security/compliance context | `Confidential` |
| `ExpirationDate` | Temporary resource cleanup | `2026-06-30` |
| `CreatedBy` | Audit trail | `terraform` or `john.doe@contoso.com` |

### Enforcement
1. **Azure Policy** — `Require` tags on resource groups and subscriptions. `Inherit` tags to child resources.
2. **Deny deployment** if mandatory tags are missing (CostCenter, Environment, Owner at minimum)
3. **Compliance KPI** — track tagging compliance percentage, target 95%+
4. **Tag inheritance in Cost Management** — fills gaps without modifying actual resource tags
5. **Regular audits** — untagged resources are invisible to cost allocation
