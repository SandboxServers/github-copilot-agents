### Bicep Testing

#### Static Analysis
| Technique | Purpose |
|---|---|
| `bicep build` | Compile and catch syntax errors |
| Bicep linter | Built-in rules for best practices, naming, secure parameters |
| `bicep snapshot` | Local-only testing — generates normalized JSON to catch unintended logic changes without Azure connection |

#### Preflight Validation
Runs automatically with `validate` or `what-if` commands. ARM validates:
- Template syntax and structure
- Resource provider registration
- API version validity
- Parameter constraints (naming rules, allowed values)

Preflight errors appear in Activity Log but NOT in deployment history (deployment never started).

**Validation Levels** (Az CLI 2.76.0+ / Az PowerShell 13.4.0+):
| Level | What It Checks |
|---|---|
| **Provider** (default) | Full validation — syntax, resources, dependencies, permissions |
| **ProviderNoRbac** | Full validation minus write permissions — useful for read-only validation |
| **Template** | Static only — syntax and structure, skips preflight and permissions |

#### What-If Operations
Preview changes without deploying. Available at all scopes:
- `az deployment group what-if` / `New-AzResourceGroupDeployment -WhatIf`
- `az deployment sub what-if` / `New-AzSubscriptionDeployment -WhatIf`
- `az deployment mg what-if` / `New-AzDeployment -WhatIf` (management group)
- `az deployment tenant what-if` (tenant)

Use `--confirm-with-what-if` or `-Confirm` to preview and prompt before deploying.

#### Post-Deployment Validation
- Verify deployed resources match specification using Azure CLI / PowerShell queries
- Smoke tests against deployed endpoints
- Policy compliance checks via Defender for Cloud
