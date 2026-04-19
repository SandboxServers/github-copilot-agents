## Compliance Validation

### Policy Compliance
- **Azure Policy** — verify deployed resources comply with assigned policies
- **Defender for Cloud** — check Secure Score and regulatory compliance dashboard
- **Resource Graph queries** — programmatic validation of resource configurations
- **Custom compliance checks** — scripts that verify specific configurations (NSG rules, encryption settings, managed identity assignments)

### Infrastructure Drift Detection
1. Run `terraform plan` on schedule (not just during deployment) to detect drift
2. Compare Bicep what-if output against expected steady state
3. Monitor Azure Policy compliance status for real-time drift detection
4. Use Azure Resource Graph to query for resources that don't match expected tags/configurations
