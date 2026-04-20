# Parameter Files

## .bicepparam Files (Preferred)
```bicep
// main.bicepparam — type-safe, validated against the Bicep file
using './main.bicep'

param location = 'eastus2'
param environment = 'prod'
param project = 'myapp'

// Complex object parameters
param networkConfig = {
  vnetName: 'vnet-prod-01'
  addressSpace: '10.0.0.0/16'
  subnets: [
    { name: 'snet-app', addressPrefix: '10.0.1.0/24' }
    { name: 'snet-data', addressPrefix: '10.0.2.0/24' }
    { name: 'snet-pe', addressPrefix: '10.0.3.0/24' }
  ]
}

param tags = {
  Environment: 'prod'
  Project: 'myapp'
  CostCenter: 'IT-12345'
}
```

## Secure Parameters
```bicep
// Read from environment variable — NEVER hardcode secrets
param adminPassword = readEnvironmentVariable('ADMIN_PASSWORD')

// Key Vault reference — preferred for CI/CD
param sqlAdminPassword = az.getSecret(
  '<subscription-id>'        // Subscription ID
  '<key-vault-rg>'           // Resource group containing the Key Vault
  '<key-vault-name>'         // Key Vault name
  '<secret-name>'            // Secret name
)

// Key Vault reference with version
param apiKey = az.getSecret(
  subscriptionId
  kvResourceGroup
  kvName
  'api-key'
  'a1b2c3d4e5f6'             // Optional: specific secret version
)
```

## Variables in Parameter Files
```bicep
using './main.bicep'

// Variables can be used for computed values
var prefix = 'myapp'
var env = 'prod'
var region = 'eastus2'

param project = prefix
param environment = env
param location = region
param resourceGroupName = '${prefix}-${env}-rg'
param storageAccountName = 'st${prefix}${env}'
```

## Extending Parameter Files (Bicep 0.28+)
```bicep
// base.bicepparam — shared defaults
using none    // No Bicep file validation for base files

param location = 'eastus2'
param project = 'myapp'
param tags = {
  Project: 'myapp'
  ManagedBy: 'Bicep'
}
```

```bicep
// dev.bicepparam — extends base with dev-specific values
using './main.bicep'
extends 'base.bicepparam'

param environment = 'dev'
param skuName = 'Basic'
// location, project, tags inherited from base
```

```bicep
// prod.bicepparam — extends base with prod-specific values
using './main.bicep'
extends 'base.bicepparam'

param environment = 'prod'
param skuName = 'Standard'
param enableHighAvailability = true
// location, project, tags inherited from base
```

## Environment-Based File Organization
```
project/
├── main.bicep
├── parameters/
│   ├── base.bicepparam          # Shared defaults
│   ├── dev.bicepparam           # Dev overrides
│   ├── staging.bicepparam       # Staging overrides
│   └── prod.bicepparam          # Prod overrides
└── modules/
    └── ...
```

## JSON Parameter Files (Legacy — Still Supported)
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": { "value": "eastus2" },
    "environment": { "value": "prod" },
    "adminPassword": {
      "reference": {
        "keyVault": {
          "id": "/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.KeyVault/vaults/{kv-name}"
        },
        "secretName": "admin-password"
      }
    }
  }
}
```

## Deployment Commands
```bash
# Deploy with .bicepparam (preferred)
az deployment group create \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters parameters/prod.bicepparam

# Deploy with JSON parameters
az deployment group create \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters @parameters/prod.parameters.json

# Override specific params on CLI
az deployment group create \
  --resource-group rg-myapp-prod \
  --template-file main.bicep \
  --parameters parameters/prod.bicepparam \
  --parameters environment='staging'    # CLI override wins

# Generate parameter file from template
az bicep generate-params --file main.bicep --output-format bicepparam
```
