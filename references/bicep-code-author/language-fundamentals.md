## Bicep Language Fundamentals

### File Structure
A Bicep file consists of these elements in any order:
- `targetScope` — deployment scope (default: `resourceGroup`)
- `metadata` — file metadata
- `import` — compile-time imports from other Bicep files or providers
- `param` — input parameters
- `var` — variables (compile-time constants)
- `func` — user-defined functions
- `type` — user-defined data types
- `resource` — resource declarations
- `module` — module references
- `output` — output values

### Resource Declarations
```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: storageAccountName
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
  }
}
```

**Key patterns:**
- Symbolic name (`storageAccount`) used for references — not the Azure resource name
- API version pinned in the resource type string
- Use `existing` keyword to reference without redeploying: `resource kv 'Microsoft.KeyVault/vaults@2023-07-01' existing = { name: kvName }`
- Child resources use `parent` property or nested syntax
- Use `dependsOn` only when Bicep can't infer implicit dependencies

### Decorators
| Decorator | Applies To | Purpose |
|---|---|---|
| `@description()` | param, var, resource, module, output, type, func | Documentation |
| `@secure()` | param, output | Mark as sensitive — never in logs or outputs |
| `@allowed()` | param | Restrict to specific values |
| `@minLength()` / `@maxLength()` | param, output, type | String/array length constraints |
| `@minValue()` / `@maxValue()` | param, output, type | Numeric range constraints |
| `@metadata()` | param, output, type | Additional metadata |
| `@export()` | type, var, func | Enable compile-time import from other files |
| `@batchSize()` | resource, module | Sequential deployment in loops |
| `@discriminator()` | param, type, output | Tagged union type discriminator |

### User-Defined Types
```bicep
type storageAccountConfigType = {
  @description('Storage account name')
  name: string

  @description('SKU name')
  @allowed(['Standard_LRS', 'Standard_GRS', 'Standard_ZRS'])
  skuName: string

  @description('Optional tags')
  tags: object?
}
```

**Union types** for constrained values:
```bicep
type environment = 'dev' | 'test' | 'prod'
type mixedArray = ('fizz' | 42 | {an: 'object'} | null)[]
```

### Compile-Time Imports
```bicep
// Import specific exports
import { storageAccountConfigType, validateName } from 'shared-types.bicep'

// Import all as namespace
import * as sharedTypes from 'shared-types.bicep'

// Import from AVM common types
import { roleAssignmentType } from 'br/public:avm/utl/types/avm-common-types:0.4.0'
```

Requires `@export()` decorator on the source. Enables language version 2.0 code generation.

### User-Defined Functions
```bicep
@export()
func buildResourceName(prefix string, suffix string) string => '${prefix}-${suffix}'
```
