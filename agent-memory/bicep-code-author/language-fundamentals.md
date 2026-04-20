# Language Fundamentals

## Type System
```bicep
// Primitive types
param name string
param count int
param enabled bool

// Array types
param subnetNames string[]
param ports int[]

// Object type
param config object

// Union types (allowed values)
param environment 'dev' | 'staging' | 'prod'
param sku 'Basic' | 'Standard' | 'Premium'

// Nullable
param optionalTag string?
```

## User-Defined Types
```bicep
// Simple type alias with constraints
@minLength(3)
@maxLength(24)
type storageAccountName = string

// Complex object type
type subnetConfigType = {
  @description('Subnet name')
  name: string

  @description('CIDR address prefix')
  addressPrefix: string

  @description('Optional NSG resource ID')
  nsgId: string?

  @description('Optional route table ID')
  routeTableId: string?

  @description('Service endpoints to enable')
  serviceEndpoints: string[]?
}

// Array of typed objects
param subnets subnetConfigType[]

// Discriminated union
@discriminator('kind')
type firewallRuleType = httpRuleType | networkRuleType

type httpRuleType = {
  kind: 'http'
  port: int
  path: string
}

type networkRuleType = {
  kind: 'network'
  port: int
  protocol: 'TCP' | 'UDP'
}
```

## User-Defined Functions
```bicep
// Simple function
func buildResourceName(project string, environment string, resourceType string) string =>
  '${project}-${environment}-${resourceType}'

// Function with conditional logic
func getSkuName(environment string) string =>
  environment == 'prod' ? 'Standard' : 'Basic'

// Function using built-in functions
func generateUniqueName(prefix string, rgId string) string =>
  toLower('${take(prefix, 11)}${uniqueString(rgId)}')

// Usage
var rgName = buildResourceName(project, environment, 'rg')
var storageName = generateUniqueName('st', resourceGroup().id)
```

## Decorators
```bicep
@description('The Azure region for all resources.')
@metadata({ example: 'eastus2' })
param location string = resourceGroup().location

@description('Environment name.')
@allowed(['dev', 'staging', 'prod'])
param environment string

@description('Administrator password.')
@secure()
@minLength(12)
param adminPassword string

@description('Number of VM instances.')
@minValue(1)
@maxValue(10)
param instanceCount int = 1

@description('Storage account name.')
@minLength(3)
@maxLength(24)
param storageAccountName string
```

## Conditional Resources
```bicep
@description('Deploy a diagnostic storage account?')
param deployDiagnostics bool = true

// Conditional deployment
resource diagnosticStorage 'Microsoft.Storage/storageAccounts@2023-05-01' = if (deployDiagnostics) {
  name: diagStorageName
  location: location
  tags: commonTags
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}

// Conditional output — use ? for resources that may not exist
output diagnosticStorageId string = deployDiagnostics ? diagnosticStorage.id : ''
```

## Loops
```bicep
// Loop over array
param subnetConfigs subnetConfigType[]

resource subnets 'Microsoft.Network/virtualNetworks/subnets@2024-05-01' = [for (config, index) in subnetConfigs: {
  name: config.name
  properties: {
    addressPrefix: config.addressPrefix
  }
}]

// Loop with index
resource nsgRules 'Microsoft.Network/networkSecurityGroups/securityRules@2024-05-01' = [for (rule, i) in securityRules: {
  name: rule.name
  properties: {
    priority: 100 + (i * 10)
    direction: rule.direction
    access: rule.access
    protocol: rule.protocol
    sourcePortRange: rule.sourcePortRange
    destinationPortRange: rule.destinationPortRange
    sourceAddressPrefix: rule.sourceAddressPrefix
    destinationAddressPrefix: rule.destinationAddressPrefix
  }
}]

// Loop with condition (filtered loop)
resource publicIps 'Microsoft.Network/publicIPAddresses@2024-05-01' = [for vm in vmConfigs: if (vm.publicIp) {
  name: '${vm.name}-pip'
  location: location
  tags: commonTags
  sku: { name: 'Standard' }
  properties: {
    publicIPAllocationMethod: 'Static'
  }
}]

// Object loop with for-in
var subnetsOutput = [for (config, i) in subnetConfigs: {
  name: config.name
  id: subnets[i].id
}]
```

## Existing Resources
```bicep
// Reference a resource that already exists — DO NOT redeploy it
resource existingVnet 'Microsoft.Network/virtualNetworks@2024-05-01' existing = {
  name: vnetName
  scope: resourceGroup(networkRgName)    // Can be in a different resource group
}

// Use its properties
output vnetId string = existingVnet.id
output vnetAddressSpace string = existingVnet.properties.addressSpace.addressPrefixes[0]

// Reference Key Vault for secret retrieval
resource existingKeyVault 'Microsoft.KeyVault/vaults@2023-07-01' existing = {
  name: keyVaultName
  scope: resourceGroup(keyVaultRgName)
}
```

## Expressions and Operators
```bicep
// Ternary
var skuName = environment == 'prod' ? 'Standard_GRS' : 'Standard_LRS'

// Null coalescing (use ?? for nullable)
var displayName = customName ?? '${project}-${environment}'

// String interpolation
var resourceName = '${project}-${environment}-${resourceType}'

// Multi-line strings
var script = '''
#!/bin/bash
echo "Hello World"
apt-get update -y
'''

// Safe dereference for optional properties
var nsgId = subnet.?nsgId ?? ''

// Spread operator (Bicep 0.28+)
var baseConfig = { location: location, tags: commonTags }
var storageConfig = { ...baseConfig, kind: 'StorageV2' }
```

## Lambda Functions
```bicep
// map — transform each element
var upperNames = map(vmNames, name => toUpper(name))

// filter — keep elements matching condition
var prodVms = filter(vmConfigs, config => config.environment == 'prod')

// reduce — aggregate values
var totalCores = reduce(vmSizes, 0, (total, vm) => total + vm.cores)

// sort
var sortedNames = sort(vmNames, (a, b) => a < b)

// flatten — collapse nested arrays
var allSubnets = flatten(map(vnets, vnet => vnet.subnets))
```
