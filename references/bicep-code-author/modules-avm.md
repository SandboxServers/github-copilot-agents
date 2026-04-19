## Modules

### Local Modules
```bicep
module storage './modules/storage.bicep' = {
  name: 'storageDeployment'
  params: {
    storageAccountName: 'mystorageaccount'
    location: location
  }
}
```

### Registry Modules (Bicep Registry)
```bicep
// Public registry (Azure Verified Modules)
module keyVault 'br/public:avm/res/key-vault/vault:0.11.0' = {
  name: 'kvDeployment'
  params: {
    name: keyVaultName
    enablePurgeProtection: true
  }
}

// Private registry
module myModule 'br:myregistry.azurecr.io/bicep/modules/storage:v1.0' = {
  name: 'storageDeployment'
  params: { ... }
}
```

### Template Specs
```bicep
module templateSpecModule 'ts:subscriptionId/resourceGroupName/templateSpecName:version' = {
  name: 'templateSpecDeploy'
  params: { ... }
}
```

### When to Use Registry vs Local
| Use Registry When | Use Local When |
|---|---|
| Module is shared across teams/repos | Module is specific to this deployment |
| Versioning and release cycle needed | Rapid iteration, not yet stable |
| AVM module exists that fits | Custom logic needed beyond AVM |
| Organization wants centralized governance | Small project, low overhead preferred |

## Azure Verified Modules (AVM)

### What AVM Provides
- Production-ready, Well-Architected-aligned modules
- Security and reliability defaults out of the box
- Versioned in the public Bicep registry (`br/public:avm/...`)
- Three categories: `res` (resource), `ptn` (pattern), `utl` (utility)
- User-defined types for code completion and type safety
- Comprehensive test coverage and usage examples

### Using AVM Effectively
- Enable `use-recent-module-versions` linter rule in `bicepconfig.json`
- Import UDTs from AVM common types for consistent typing
- Override only what your environment requires — defaults are WAF-aligned
- Check module documentation for all parameters and usage examples
- Use role assignment name references instead of GUIDs

### When to Write Custom
- No AVM module exists for the resource
- AVM module doesn't support needed configuration
- Need tighter control over resource properties
- Organizational standards conflict with AVM defaults
