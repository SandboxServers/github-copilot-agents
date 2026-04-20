# Import, Export & Compile-Time Sharing

Bicep's `import`/`@export()` system enables sharing types, variables, and functions across files without module deployments. Compile-time imports automatically enable language version 2.0 code generation.

## Exporting Symbols
Use `@export()` on `type`, `var`, and `func` statements to make them importable:
```bicep
// shared/types.bicep

@export()
@description('Standard tag structure for all resources.')
type tagsType = {
  @description('Environment name.')
  environment: 'dev' | 'staging' | 'prod'
  @description('Cost center code.')
  costCenter: string
  @description('Owning team.')
  team: string
}

@export()
@description('Subnet configuration for VNet modules.')
type subnetConfigType = {
  name: string
  addressPrefix: string
  nsgId: string?
  serviceEndpoints: string[]?
}

@export()
func buildResourceName(project string, env string, suffix string) string =>
  '${project}-${env}-${suffix}'

@export()
var defaultLocation = 'eastus2'
```

**Rules:**
- Only `type`, `var`, and `func` can be exported — not `param`, `resource`, or `output`
- Exported variables must be compile-time constants (no runtime functions like `resourceGroup()`)
- Place `@export()` directly above the statement

## Importing Symbols
```bicep
// Named imports — import specific symbols
import { tagsType, subnetConfigType } from './shared/types.bicep'

param tags tagsType
param subnets subnetConfigType[]

// Aliased imports — rename to avoid conflicts
import { buildResourceName as nameBuilder } from './shared/types.bicep'
var rgName = nameBuilder(project, environment, 'rg')

// Wildcard imports — import everything under a namespace
import * as shared from './shared/types.bicep'
param tags shared.tagsType
var loc = shared.defaultLocation
```

## Importing from Registry Modules
You can import types from published Bicep registry modules without deploying them:
```bicep
// Import types from a public registry module
import { roleAssignmentType } from 'br/public:avm/utl/types/avm-common-types:0.5.0'

@description('Optional. Role assignments to create.')
param roleAssignments roleAssignmentType[]?

// Import from private ACR
import { appConfigType } from 'br:myregistry.azurecr.io/bicep/types/common:1.0.0'
```

## File Organization Pattern
```
project/
├── main.bicep                  # Orchestrator — imports types, calls modules
├── main.bicepparam
├── shared/
│   ├── types.bicep             # Exported UDTs (tags, subnet configs, etc.)
│   ├── naming.bicep            # Exported naming functions
│   └── constants.bicep         # Exported compile-time variables
└── modules/
    ├── networking.bicep         # Imports from shared/, defines resources
    ├── storage.bicep
    └── app-service.bicep
```

## Import vs Module — When to Use Each
| Need | Use |
|------|-----|
| Share a type definition across files | `import { myType } from './types.bicep'` |
| Share a helper function | `import { myFunc } from './helpers.bicep'` |
| Share a constant value | `import { myConst } from './constants.bicep'` |
| Deploy an Azure resource | `module myMod './modules/resource.bicep'` |
| Reuse a deployment pattern | `module myMod 'br:registry/path:version'` |

Imports are **compile-time only** — they add no nested deployments to the ARM template. Modules create nested deployments.

## Gotchas
- `@export()` on a `var` that uses `resourceGroup()` or `subscription()` will fail — these are runtime functions
- Do NOT place an `import` block between a parameter's decorator and its declaration — it breaks decorator binding
- Wildcard imports (`import *`) bring everything — prefer named imports for clarity
- Circular imports between files are not allowed
- Import paths are relative to the importing file, not the project root
