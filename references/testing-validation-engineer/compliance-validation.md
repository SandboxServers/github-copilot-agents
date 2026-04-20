## Compliance Validation

### Azure Policy Compliance Testing

- Verify deployed resources comply with assigned Azure Policy definitions
- Use `Get-AzPolicyState` or `az policy state list` to check compliance programmatically
- Test deny policies by attempting non-compliant deployments — expect the deployment to fail
- Test audit policies by deploying resources and verifying compliance state is flagged
- Run policy compliance checks as a post-deployment pipeline stage
- Test custom policy definitions in isolated subscriptions before assigning broadly

### Resource Deployment Validation

After every deployment, validate that resources match the intended specification:

| What to Validate | How to Check |
|-----------------|--------------|
| **SKU and tier** | `az resource show` — verify SKU matches expected value |
| **Region/location** | Confirm resources deployed to intended region |
| **Networking rules** | Verify NSG rules, private endpoints, VNET integration |
| **Encryption settings** | Check encryption at rest and in transit configuration |
| **Identity configuration** | Verify managed identity assignments and RBAC roles |
| **Diagnostic settings** | Confirm logs and metrics route to correct destinations |
| **Tags** | Validate required tags (environment, cost center, owner) are present |

- Script all validations and run as automated post-deployment checks
- Use Azure Resource Graph for efficient bulk queries across resource groups

### Post-Deployment Checks

- Health endpoint validation — HTTP 200 from application health routes
- Connectivity checks — application can reach databases, caches, external APIs
- Configuration verification — feature flags, connection strings, app settings are correct
- Certificate validation — TLS certificates are valid and not near expiration
- DNS resolution — custom domains resolve to correct endpoints
- Automate all checks — post-deployment validation should be a pipeline stage, not a manual step

### Drift Detection

- Run `terraform plan` on a schedule to detect infrastructure drift (not just during deployment)
- Compare Bicep `what-if` output against expected steady state periodically
- Monitor Azure Policy compliance status continuously — non-compliant resources indicate drift
- Use Azure Resource Graph queries to find resources missing required tags or configurations
- Alert on drift detection — treat drift as an incident requiring remediation
- Common drift sources: manual portal changes, out-of-band scripts, auto-scaling side effects

### Configuration Audit

- Periodically audit resource configurations against documented standards
- Compare actual configurations to infrastructure-as-code source of truth
- Track configuration changes using Azure Activity Log and Change Analysis
- Verify RBAC assignments match the principle of least privilege
- Audit network security group rules against approved baselines
- Export audit results for compliance reporting and evidence collection

### Regulatory Compliance Validation Scripts

- Automate checks for regulatory frameworks: SOC 2, ISO 27001, HIPAA, PCI-DSS, FedRAMP
- Use Microsoft Defender for Cloud regulatory compliance dashboard as a starting point
- Script custom checks for controls not covered by built-in policies
- Run compliance validation on a schedule (weekly or after each deployment)
- Generate compliance evidence reports automatically for auditors
- Version control compliance scripts and review them alongside infrastructure code
- Map each automated check to a specific compliance control ID for traceability
