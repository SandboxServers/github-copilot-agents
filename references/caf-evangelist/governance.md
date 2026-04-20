## Cloud Governance

### The Five Governance Disciplines

Cloud governance ensures the cloud environment remains controlled, compliant, and
cost-effective. CAF defines five disciplines that together provide comprehensive governance.

#### 1. Cost Management

Control cloud spending and enable financial accountability.

- **Budgets** — set spending limits per subscription, resource group, or management group
- **Tagging** — enforce cost allocation tags (CostCenter, Owner, Environment, Application)
- **Alerts** — notify when spending approaches or exceeds thresholds
- **Chargeback/showback** — attribute costs to business units or teams
- **Azure tools** — Cost Management, Advisor cost recommendations, budgets API
- **Regular reviews** — monthly cost reviews with stakeholders, identify anomalies

#### 2. Security Baseline

Establish minimum security controls across all cloud resources.

- **Microsoft Cloud Security Benchmark (MCSB)** — baseline security controls
- **Microsoft Defender for Cloud** — posture management, threat protection, Secure Score
- **Microsoft Sentinel** — SIEM/SOAR for security operations
- **Regulatory compliance** — built-in compliance dashboards (ISO 27001, SOC 2, NIST)
- **Vulnerability management** — continuous scanning, remediation tracking
- **Security policies** — enforce encryption, disable public endpoints, require HTTPS

#### 3. Identity Baseline

Control authentication, authorization, and privileged access.

- **Microsoft Entra ID** — centralized identity provider, SSO
- **Conditional access** — risk-based access policies (location, device, sign-in risk)
- **Privileged Identity Management (PIM)** — just-in-time, time-limited admin access
- **RBAC** — least-privilege role assignments, avoid Owner at broad scopes
- **Managed identities** — eliminate credential management for service-to-service auth
- **Emergency access** — break-glass accounts with monitoring and alerts

#### 4. Resource Consistency

Enforce standards that keep the environment organized and predictable.

- **Naming conventions** — consistent resource names (e.g., `rg-app-prod-eastus`)
- **Tagging standards** — mandatory tags enforced via policy
- **Allowed regions** — restrict deployments to approved Azure regions
- **Approved SKUs** — limit VM sizes, storage tiers to approved list
- **Resource locks** — protect critical resources from accidental deletion
- **Management groups** — organize subscriptions by function, environment, or business unit

#### 5. Deployment Acceleration

Standardize how resources are deployed and configured.

- **Infrastructure as Code (IaC)** — Bicep, Terraform, or ARM templates for all deployments
- **CI/CD pipelines** — automated testing, approval gates, deployment orchestration
- **Policy-as-code** — Azure Policy definitions managed in source control
- **Template repositories** — approved modules and patterns for common architectures
- **Configuration management** — desired state configuration, drift detection
- **Environment promotion** — dev → staging → production with consistent IaC

### Governance Maturity

#### Minimum Viable Governance (MVG)

Start with essential controls rather than trying to implement everything at once:

- Basic cost alerting and budget limits
- Required tagging policy (audit mode initially)
- Allowed regions and VM SKU restrictions
- RBAC with no broad Owner assignments
- Diagnostic settings deployed automatically

#### Iterative Improvement

Mature governance over time based on actual risks and incidents:

```
MVG (Day 1)
├─ Basic policies (audit mode)
├─ Cost alerts
└─ RBAC baseline

Wave 2 (Month 2-3)
├─ Policies moved from audit → deny
├─ Tagging enforcement
└─ Defender for Cloud enabled

Wave 3 (Month 4-6)
├─ PIM for privileged roles
├─ Sentinel deployed
└─ Compliance dashboards

Ongoing
├─ Policy refinement based on incidents
├─ New policies for new resource types
└─ Governance reviews quarterly
```

### Azure Policy as Enforcement Mechanism

Azure Policy is the primary governance enforcement tool:

| Policy Effect | Behavior | Use Case |
|--------------|----------|----------|
| **Deny** | Block non-compliant creation | Enforce allowed regions, required tags |
| **Audit** | Log violation, allow creation | Assess compliance before enforcing |
| **DeployIfNotExists** | Auto-create companion resource | Enable diagnostics, deploy monitoring |
| **Modify** | Change resource properties | Add tags, enable encryption settings |
| **Append** | Add properties to resource | Add IP restrictions, add tags |

Best practice: start with Audit, graduate to Deny after validating impact. Use policy
initiatives (sets of policies) rather than individual policy assignments for manageability.
