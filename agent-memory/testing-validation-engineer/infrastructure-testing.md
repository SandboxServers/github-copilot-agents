## Infrastructure Testing

### Terraform Testing

#### Static Analysis

| Tool | Purpose | Speed |
|------|---------|-------|
| `terraform validate` | Syntax and internal consistency | Seconds |
| `terraform fmt` | Format compliance | Seconds |
| **tflint** (azurerm ruleset) | Linting — deprecated arguments, invalid values, naming | Seconds |
| **Checkov** | Security and compliance policy scanning (CIS, NIST, PCI-DSS) | Seconds |

#### Plan Validation

- `terraform plan` — preview changes, catch unexpected destroys, identify drift
- **terraform-compliance** — BDD-style policy testing against plan output
- Plan assertions — programmatic inspection of plan JSON for expected resource counts
- **Infracost** — estimate cost impact of planned changes before apply

#### Terraform Test Framework

- Native `terraform test` command — HCL-based tests that deploy, validate, and destroy
- Tests run in isolated environments to avoid impacting real infrastructure
- Assert on resource attributes, output values, and data source results
- Runs as part of CI pipeline — fail the build on test failure

#### Terratest (Go)

- Full lifecycle testing: deploy with Terraform, validate with Azure SDK calls, destroy
- Best for complex validation that requires programmatic inspection
- Supports retry logic for eventual consistency in cloud resources
- Parallel test execution for faster feedback

### Bicep Testing

#### What-If Operations

- `az deployment group what-if` — preview changes without deploying
- Available at all scopes: resource group, subscription, management group, tenant
- Use `--confirm-with-what-if` to preview and prompt before deploying
- Integrate into PR validation to show reviewers what will change

#### ARM-TTK (Template Test Toolkit)

- Validates ARM/Bicep templates against best practices
- Checks parameter naming, secure parameter usage, API versions
- Run as part of CI to catch template issues early

#### Pester for Deployment Validation

- Post-deployment validation using PowerShell and Pester
- Query deployed resources with `Get-AzResource` and assert configuration
- Verify NSG rules, encryption settings, managed identity assignments
- Publish test results as NUnit XML for pipeline consumption

### Policy Testing

#### Azure Policy Compliance

- Verify deployed resources comply with assigned policies
- Use `Get-AzPolicyState` to check compliance status programmatically
- Test policy definitions in isolated subscriptions before broad assignment
- Validate deny policies by attempting non-compliant deployments (expect failure)

#### Resource Deployment Validation

- After every deployment, verify resources match the specification
- Check SKUs, regions, networking rules, identity configuration
- Use Azure Resource Graph for efficient bulk queries across resources
- Script validations and run as post-deployment pipeline stage

### Smoke Tests After Deployment

- Hit health check endpoints immediately after deployment
- Verify DNS resolution, TLS certificates, port connectivity
- Check application can reach its dependencies (database, cache, APIs)
- Validate configuration values are correct (connection strings, feature flags)
- Automated smoke tests should block promotion to next environment on failure

### Canary Deployments

- Route a small percentage of traffic to the new version first
- Monitor error rates, latency, and resource consumption during canary window
- Automated rollback if metrics degrade beyond threshold
- Gradually increase traffic percentage as confidence grows
- Use Azure Traffic Manager, Front Door, or Kubernetes canary strategies
