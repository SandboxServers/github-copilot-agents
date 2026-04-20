# Bicep Gotchas & Surprising Behaviors

Things that are technically correct but behave unexpectedly.

## Circular Dependency Detection
Bicep creates implicit dependencies when you reference one resource's symbolic name in another. Cycles cause `CircularDependencyDetected` errors.
```bicep
// PROBLEM — webapp1 needs webapp2's hostname, webapp2 needs webapp1's hostname
// FIX — break the cycle by moving config into child resources
resource webapp1 'Microsoft.Web/sites@2023-12-01' = { name: 'app1', ... }
resource webapp2 'Microsoft.Web/sites@2023-12-01' = { name: 'app2', ... }
// Deploy config separately with dependsOn on BOTH apps
resource app1Config 'Microsoft.Web/sites/config@2023-12-01' = {
  parent: webapp1
  name: 'appsettings'
  dependsOn: [webapp2]
  properties: { PARTNER_URL: 'https://${webapp2.properties.defaultHostName}' }
}
```

## Deployment Ordering Is Not Guaranteed
Bicep/ARM deploys independent resources in parallel with **no guaranteed order** unless there is a dependency chain. Don't assume resource A completes before resource B just because it appears first in the file.
```bicep
// These may deploy in ANY order — use dependsOn or implicit refs to control
resource nsg 'Microsoft.Network/networkSecurityGroups@2024-05-01' = { ... }
resource vnet 'Microsoft.Network/virtualNetworks@2024-05-01' = { ... }
```

## String Interpolation in Conditions
String interpolation inside conditions can produce surprising results when the interpolated value comes from a resource that may not exist.
```bicep
// GOTCHA — the interpolation is evaluated even when condition is false
// If sqlServer doesn't exist, this may error during template evaluation
output connString string = deploySQL ? 'Server=${sqlServer.properties.fullyQualifiedDomainName}' : ''

// SAFER — use conditional output referencing the module output
output connString string = deploySQL ? sqlModule.outputs.connectionString : ''
```

## The `existing` Keyword Limitations
- Only works for resources that ARM can GET (most resource types, but not all child resources)
- Does NOT validate that the resource actually exists at deploy time — deployment succeeds, but downstream references may fail
- Cannot use `existing` with `dependsOn` to order against a resource in a different deployment
- Cross-resource-group references require `scope: resourceGroup('other-rg')`

## Module Outputs Are Always Evaluated
When a module has `condition: false`, its outputs are still type-checked. Referencing a conditional module's outputs requires the ternary pattern:
```bicep
module sql './modules/sql.bicep' = if (deploySQL) { ... }

// GOTCHA — this compiles but returns empty/null if condition is false
// You MUST guard the output access
output sqlFqdn string = deploySQL ? sql.outputs.fqdn : ''
```

## What-If False Positives
What-if commonly reports phantom changes that don't actually exist:
- **Tag ordering** — reports tag changes when only the JSON key order differs
- **Password/secret fields** — always shows "Modified" because ARM redacts values
- **Nested deployments with runtime functions** — what-if short-circuits and shows `NoChange` for module contents when `reference()` is used in the compiled ARM template
- **Array property ordering** — subnets, IP configs, NSG rules may show as modified

**Mitigation:** Review what-if output critically. Focus on resource creates/deletes, not property-level noise.

## Deployment Stack: Delete vs Detach
When removing a resource from a deployment stack template:
- `actionOnUnmanage: 'deleteAll'` — **physically deletes** the resource from Azure
- `actionOnUnmanage: 'detachAll'` — removes from stack management but **leaves resource alive**
- Default behavior varies by resource type — always set this explicitly
- Deny settings on stacks can block manual fixes if misconfigured

## Parameter File Precedence
When the same parameter is set in multiple places:
1. **CLI `--parameters` inline** (highest priority) → overrides everything
2. **`.bicepparam` file values** → override template defaults
3. **Template `param` defaults** (lowest priority)

```bash
# CLI value wins over .bicepparam value for 'environment'
az deployment group create --template-file main.bicep \
  --parameters main.bicepparam --parameters environment=prod
```

## Private Module Registry Authentication
- ACR-hosted modules require `az acr login` or a configured credential helper
- Token expiry causes cryptic "module not found" errors — not auth errors
- CI/CD pipelines need explicit `az login` + `az acr login` before `bicep build`
- `bicepconfig.json` aliases don't store credentials — auth is handled by the environment

## Resource API Version Pitfalls
- **Too new (preview):** Preview API versions may change without notice, breaking deployments
- **Too old:** Missing properties that were added in newer versions — deployment succeeds but config is ignored
- **Recommendation:** Use the latest GA (stable) API version. Only use preview when a feature requires it, and pin the version explicitly.

```bicep
// GOOD — latest GA
resource kv 'Microsoft.KeyVault/vaults@2023-07-01' = { ... }

// RISKY — preview API
resource kv 'Microsoft.KeyVault/vaults@2024-11-01-preview' = { ... }
```

## ARM Template Hard Limits
These limits apply to the **compiled ARM JSON**, not the Bicep source:
| Limit | Value |
|-------|-------|
| Parameters | 256 |
| Variables | 256 |
| Resources (including copy count) | 800 |
| Outputs | 64 |
| Template size | 4 MB |
| Parameter file size | 4 MB |
| Deployment name length | 64 characters |
| Nested deployment depth | 5 levels |

AVM modules inflate compiled template size significantly. Monitor with `bicep build` and check the output JSON size. Use loops instead of repeated module references to reduce compiled size.
