# Testing

## Native Terraform Tests (.tftest.hcl)
Available since Terraform 1.6. Preferred over external testing frameworks.

### Unit Test (Plan Only)
```hcl
# tests/unit/naming.tftest.hcl
variables {
  project     = "myapp"
  environment = "dev"
  location    = "eastus2"
}

run "resource_naming" {
  command = plan    # No infrastructure created

  assert {
    condition     = azurerm_resource_group.main.name == "myapp-dev-rg"
    error_message = "Resource group name should follow naming convention."
  }

  assert {
    condition     = azurerm_resource_group.main.location == "eastus2"
    error_message = "Location should be eastus2."
  }
}
```

### Integration Test (Apply + Destroy)
```hcl
# tests/integration/deploy.tftest.hcl
variables {
  project     = "tftest"
  environment = "test"
  location    = "eastus2"
}

# Override provider for test isolation
provider "azurerm" {
  features {}
  subscription_id = "test-subscription-id"
}

run "deploy_infrastructure" {
  command = apply    # Actually creates resources

  assert {
    condition     = azurerm_resource_group.main.id != ""
    error_message = "Resource group should be created."
  }

  assert {
    condition     = azurerm_virtual_network.main.address_space[0] == "10.0.0.0/16"
    error_message = "VNet address space should be 10.0.0.0/16."
  }

  # Resources are automatically destroyed after the test
}
```

### Testing Variable Validation (Expected Failures)
```hcl
run "invalid_environment_rejected" {
  command = plan

  variables {
    environment = "invalid"
  }

  expect_failures = [
    var.environment    # Should fail validation
  ]
}

run "invalid_cidr_rejected" {
  command = plan

  variables {
    vnet_cidr = "not-a-cidr"
  }

  expect_failures = [
    var.vnet_cidr
  ]
}
```

### Testing Module Outputs
```hcl
run "module_outputs" {
  command = plan

  assert {
    condition     = output.resource_group_name != ""
    error_message = "Resource group name output should not be empty."
  }

  assert {
    condition     = can(regex("^/subscriptions/", output.resource_group_id))
    error_message = "Resource group ID should be a valid Azure resource ID."
  }
}
```

### Mock Providers (TF 1.7+)
```hcl
# Mock the azurerm provider for pure unit tests — no Azure connection needed
mock_provider "azurerm" {
  mock_data "azurerm_client_config" {
    defaults = {
      tenant_id       = "00000000-0000-0000-0000-000000000000"
      subscription_id = "00000000-0000-0000-0000-000000000000"
      object_id       = "00000000-0000-0000-0000-000000000000"
    }
  }
}

run "unit_test_with_mocks" {
  command = plan
  
  assert {
    condition     = azurerm_key_vault.main.tenant_id != ""
    error_message = "Key Vault should have a tenant ID."
  }
}
```

### Test File Organization
```
project/
├── main.tf
├── variables.tf
├── outputs.tf
└── tests/
    ├── unit/
    │   ├── naming.tftest.hcl
    │   ├── validation.tftest.hcl
    │   └── outputs.tftest.hcl
    └── integration/
        ├── deploy.tftest.hcl
        └── fixtures/
            └── test.tfvars
```

```bash
# Run all tests
terraform test

# Run specific test file
terraform test -filter=tests/unit/naming.tftest.hcl

# Verbose output
terraform test -verbose
```

## Tooling Stack

### tflint
```hcl
# .tflint.hcl
plugin "azurerm" {
  enabled = true
  version = "0.27.0"
  source  = "github.com/terraform-linters/tflint-ruleset-azurerm"
}

rule "terraform_naming_convention" {
  enabled = true
  format  = "snake_case"
}

rule "terraform_documented_variables" {
  enabled = true
}

rule "terraform_documented_outputs" {
  enabled = true
}

rule "terraform_typed_variables" {
  enabled = true
}
```

### Checkov (Security Scanning)
```yaml
# .checkov.yaml
framework:
  - terraform
skip-check:
  - CKV_AZURE_35    # Skip specific checks with documented reason
soft-fail-on:
  - CKV_AZURE_33    # Warn but don't fail on specific checks
```

### terraform-docs
```yaml
# .terraform-docs.yml
formatter: markdown table
output:
  file: README.md
  mode: inject
sort:
  enabled: true
  by: required
```

## Pre-Commit Configuration
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/antonbabenko/pre-commit-terraform
    rev: v1.96.0
    hooks:
      - id: terraform_fmt
      - id: terraform_validate
      - id: terraform_tflint
        args:
          - --args=--config=__GIT_WORKING_DIR__/.tflint.hcl
      - id: terraform_docs
        args:
          - --hook-config=--path-to-file=README.md
          - --hook-config=--add-to-existing-file=true
      - id: terraform_checkov
        args:
          - --args=--config-file __GIT_WORKING_DIR__/.checkov.yaml
```

## CI/CD Pipeline Integration

### GitHub Actions
```yaml
name: Terraform
on:
  pull_request:
    paths: ['infra/**']

permissions:
  id-token: write
  contents: read
  pull-requests: write

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: "1.9.x"
      
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      
      - name: Terraform Init
        working-directory: infra
        run: terraform init -backend-config=backend-prod.hcl
      
      - name: Terraform Plan
        working-directory: infra
        run: terraform plan -out=tfplan -var-file=envs/prod.tfvars
      
      - name: Terraform Test
        working-directory: infra
        run: terraform test -filter=tests/unit/
```

### Azure Pipelines
```yaml
- task: TerraformInstaller@0
  inputs:
    terraformVersion: '1.9.x'

- task: AzureCLI@2
  displayName: 'Terraform Plan'
  inputs:
    azureSubscription: 'terraform-wif-connection'
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      cd infra
      terraform init -backend-config=backend-$(Environment).hcl
      terraform plan -out=tfplan -var-file=envs/$(Environment).tfvars
      terraform show -json tfplan > plan.json
    addSpnToEnvironment: true
  env:
    ARM_USE_OIDC: true
    ARM_CLIENT_ID: $(servicePrincipalId)
    ARM_TENANT_ID: $(tenantId)
    ARM_SUBSCRIPTION_ID: $(subscriptionId)
```
