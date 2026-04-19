## Parameter Files

### Bicepparam Format (Recommended)
```bicep
using './main.bicep'

param keyVaultName = 'kv-prod-001'
param enablePurgeProtection = true
param location = 'eastus2'
```

**Key Vault secret retrieval in bicepparam:**
```bicep
using './main.bicep'

param sqlServerName = 'sql-prod-001'
param adminLogin = 'sqladmin'
param adminPassword = az.getSecret('<sub-id>', '<rg-name>', '<kv-name>', '<secret-name>')
```

### JSON Parameter Files
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "keyVaultName": { "value": "kv-prod-001" },
    "adminPassword": {
      "reference": {
        "keyVault": { "id": "/subscriptions/.../Microsoft.KeyVault/vaults/myKv" },
        "secretName": "sqlAdminPassword"
      }
    }
  }
}
```

### Compile-Time vs Deploy-Time Configuration
| Compile-Time | Deploy-Time |
|---|---|
| Resolved when Bicep builds to ARM JSON | Resolved when ARM processes the template |
| `import`, `@export()`, type expressions | Parameter values, Key Vault references |
| `var` with literal values | `param` with runtime defaults (`resourceGroup().location`) |
| User-defined functions | ARM template functions (`reference()`, `listKeys()`) |
