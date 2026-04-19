## Module Architecture

### AVM (Azure Verified Modules) Standards
- Microsoft-maintained, WAF-aligned, tested modules
- Two types: **Resource modules** (single resource + extensions) and **Pattern modules** (multi-resource architectures)
- Available in Terraform Registry under `Azure/` namespace
- Use AVM when: module exists, covers your requirements, version is stable
- Write custom when: AVM doesn't exist, AVM is too restrictive, you need cross-provider composition

### Module Directory Structure (AVM Standard)
```
module-root/
├── main.tf                    # Primary resource definitions
├── main.resource1.tf          # Dot notation for larger modules
├── variables.tf               # Input variables with descriptions and validation
├── outputs.tf                 # Meaningful outputs (resource IDs, endpoints, names)
├── locals.tf                  # Computed values, naming conventions, transformations
├── terraform.tf               # Required providers, version constraints
├── _header.md                 # Documentation generation header
├── _footer.md                 # Documentation generation footer
├── README.md                  # Auto-generated (terraform-docs)
├── examples/
│   ├── default/               # Minimum viable example (required inputs only)
│   ├── complete/              # All features enabled
│   └── <scenario>/            # Specific use cases
├── modules/                   # Sub-modules (if needed)
└── tests/
    ├── unit/                  # .tftest.hcl files for unit testing
    └── integration/           # .tftest.hcl files for integration testing
```

### Module Design Principles
1. **Reusable without being over-abstracted** — expose the knobs people actually need, don't hide everything behind opinionated defaults
2. **No provider blocks in modules** — only `configuration_aliases` when needing multiple provider instances (e.g., multi-region)
3. **Meaningful defaults** — every variable should have a sensible default where possible
4. **Validation blocks** on variables that have constraints (CIDR ranges, naming patterns, enum-like values)
5. **Lifecycle rules** — use `prevent_destroy` for stateful resources, `ignore_changes` for Azure-managed properties
6. **Outputs** — always output resource ID, name, and any endpoints/connection info consumers need

### Provider Declarations in Modules (AVM Rule TFNFR27)
```hcl
# CORRECT — configuration_aliases only
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
      configuration_aliases = [azurerm.alternate]
    }
  }
}

# WRONG — provider block with configuration in module
provider "azurerm" {
  features {}  # This breaks count/for_each/depends_on for module consumers
}
```
