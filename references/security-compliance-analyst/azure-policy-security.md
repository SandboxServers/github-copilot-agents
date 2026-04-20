# Azure Policy for Security

Azure Policy is the enforcement mechanism for security requirements at scale.

## Policy Effect Types

| Effect | Behavior | Use Case |
|---|---|---|
| **Audit** | Reports non-compliance but allows deployment | Discovery phase — understand current state |
| **Deny** | Blocks non-compliant resource deployment | Enforcement — prevent insecure configurations |
| **DeployIfNotExists (DINE)** | Deploys a remediation resource if missing | Auto-remediation — add diagnostic settings, enable encryption |
| **Modify** | Adds or changes properties on a resource | Tagging, adding properties — set minimum TLS version |
| **Append** | Adds properties to a resource (legacy, prefer Modify) | Adding IP rules, tags |
| **AuditIfNotExists** | Audits when a related resource is missing | Check if diagnostic settings exist |

## Essential Security Policies

Policies every production environment should have:

- **Require HTTPS on Storage accounts** — deny creation of storage accounts with HTTP enabled
- **Require private endpoints** — audit or deny PaaS services without private endpoint connections
- **Deny public IP creation** — prevent public IP resources in production subscriptions
- **Require minimum TLS version** — enforce TLS 1.2+ on all services that support it
- **Require encryption settings** — enforce CMK for sensitive workloads, enforce infrastructure encryption
- **Deny classic resources** — block deployment of classic (ASM) resources
- **Require NSG on subnets** — audit subnets without network security group associations
- **Require diagnostic settings** — DINE policy to deploy diagnostic settings on all supported resources

## Built-In Security Initiatives

Pre-packaged policy sets for compliance frameworks:

- **CIS Microsoft Azure Foundations Benchmark** — comprehensive security baseline from Center for Internet Security
- **Microsoft Cloud Security Benchmark (MCSB)** — Microsoft's own security benchmark, the foundation for Defender for Cloud recommendations
- **NIST SP 800-53** — US government security controls framework (Rev 4 and Rev 5 available)
- **PCI DSS** — payment card industry security controls
- **HIPAA HITRUST** — healthcare data protection controls

## Assignment Strategy

Where to assign policies matters:

- **Management group level** — broad security policies that apply everywhere (TLS requirements, diagnostic settings, deny classic resources)
- **Subscription level** — workload-specific policies (require private endpoints for production subscription, audit-only for dev)
- **Resource group level** — exceptions and overrides for specific workloads
- **Use exclusions sparingly** — every exclusion is documented with justification, approved by security team, and has an expiry date

## Remediation

For policies with DINE or Modify effects:

- **Managed identity required** — DINE policies need a managed identity to create or modify resources. Assign appropriate RBAC to the identity.
- **Remediation tasks** — for existing non-compliant resources, create remediation tasks to bring them into compliance
- **New resources auto-remediated** — DINE and Modify effects apply automatically to new deployments
- **Track remediation progress** — monitor remediation task status in the Policy portal

## Custom Policies

When built-in policies don't cover your requirement:

- **Policy rule structure** — `if` condition (what to match) and `then` effect (what to do)
- **Use aliases** — aliases reference specific resource properties (e.g., `Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly`)
- **Find aliases** — `Get-AzPolicyAlias` in PowerShell or `az provider show` in CLI
- **Test in audit mode first** — always deploy custom policies as audit before switching to deny
- **Version control** — store policy definitions in source control, deploy via pipeline
- **Policy definition structure** — mode (All, Indexed), parameters, policy rule

## Best Practices

- **Start with audit** — visibility before enforcement. Understand the blast radius before blocking deployments.
- **Graduate to deny/DINE** — once teams are aware and remediation is underway, switch from audit to enforcement
- **Track compliance score** — use Azure Policy compliance dashboard. Set targets for each initiative.
- **Exemptions require security team approval** — no self-service exemptions for security policies
- **Review policy assignments quarterly** — remove obsolete policies, update to newer versions, verify effectiveness
- **Use policy as code** — define policies in JSON/Bicep, store in repo, deploy via CI/CD pipeline
- **Monitor policy evaluation** — check for policies that never trigger (may be misconfigured) or always trigger (may be too broad)
