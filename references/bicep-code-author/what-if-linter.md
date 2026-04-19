## What-If Operations

### Preview Changes Before Deploying
```bash
# Azure CLI
az deployment group what-if --resource-group myRG --template-file main.bicep

# PowerShell
New-AzResourceGroupDeployment -WhatIf -ResourceGroupName myRG -TemplateFile main.bicep
```

### Validation Levels (CLI 2.76.0+ / PS 13.4.0+)
| Level | Checks |
|---|---|
| **Provider** (default) | Full — syntax, resources, dependencies, permissions |
| **ProviderNoRbac** | Full minus write permissions |
| **Template** | Static only — syntax and structure |

### Change Types
- **Create** — new resource to be created
- **Delete** — resource exists but not in template (complete mode only)
- **Modify** — resource exists, properties will change
- **NoChange** — resource redeployed with no property changes
- **Ignore** — resource not in template, won't be touched
- **NoEffect** — read-only property, ignored by service
- **Deploy** — no changes possible to determine

### Limitations
What-if cannot resolve: `reference()`, `listKeys()`, `newGuid()`, `utcNow()`, secure parameter values, or properties from resources not in the same template.

## Linter Rules

### Key Rules to Enable Beyond Defaults
| Rule | Default | Recommendation |
|---|---|---|
| `use-recent-api-versions` | off | Enable — catch stale API versions |
| `use-recent-module-versions` | off | Enable — catch outdated AVM modules |
| `no-hardcoded-location` | off | Enable — enforce parameterized locations |
| `no-loc-expr-outside-params` | off | Enable — location expressions only in params |
| `use-resource-id-functions` | off | Enable — no hand-crafted resource IDs |

### Security-Critical Rules (Always Warning or Error)
- `secure-parameter-default` — `@secure()` params must not have defaults
- `secure-params-in-nested-deploy` — secure values in nested deployments
- `secure-secrets-in-params` — secrets handled securely
- `outputs-should-not-contain-secrets` — no secrets in outputs
- `adminusername-should-not-be-literal` — no hardcoded admin usernames
- `protect-commandtoexecute-secrets` — use protectedSettings for scripts
- `use-secure-value-for-secure-inputs` — secure values for secure resource properties

### Configuration
```json
// bicepconfig.json
{
  "analyzers": {
    "core": {
      "rules": {
        "use-recent-api-versions": { "level": "warning" },
        "use-recent-module-versions": { "level": "warning" },
        "no-hardcoded-location": { "level": "warning" }
      }
    }
  }
}
```

### Suppression
```bicep
// Suppress on next line
#disable-next-line no-hardcoded-env-urls
var endpoint = 'https://management.azure.com'

// Suppress for entire resource
#disable-diagnostics
resource legacy 'Microsoft.Resources/deployments@2021-04-01' = { ... }
```
