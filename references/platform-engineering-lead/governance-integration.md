## Governance Integration

### Platform as Governance Enforcer
The platform is the natural enforcement point for organizational governance.
Rather than relying on manual reviews and approval chains, the platform encodes
governance requirements into automated controls that apply consistently
across all teams and workloads.

### Policy-as-Code
Express governance requirements as executable code:

| Tool | Scope | Use Case |
|---|---|---|
| **Azure Policy** | Azure resources | Enforce SKU restrictions, tagging, network rules, encryption |
| **OPA/Gatekeeper** | Kubernetes clusters | Enforce pod security, resource limits, image sources |
| **Terraform Sentinel** | IaC deployments | Validate infrastructure plans before apply |
| **GitHub/ADO Branch Policies** | Source control | Enforce reviews, CI checks, commit signing |

Policy-as-code provides:
- **Consistency** — same rules apply to every team, every time
- **Auditability** — every policy evaluation is logged
- **Version control** — policies tracked in git with change history
- **Testability** — policies can be validated before deployment

### Guardrails, Not Gates
The platform should make compliant behavior easy and non-compliant behavior hard.

| Gate (Avoid) | Guardrail (Prefer) |
|---|---|
| Manual approval required for every deployment | Automated policy check approves compliant deployments |
| Security team reviews every infrastructure change | Security controls baked into golden path templates |
| Cost approval needed for each resource | Approved SKU list with automatic budget alerts |
| Compliance audit once per quarter | Continuous compliance monitoring with real-time dashboards |

Gates create bottlenecks and slow delivery. Guardrails maintain standards while
preserving developer velocity.

### Security Controls in Golden Paths
Security is not a separate step — it's embedded in every golden path:
- **Dependency scanning** runs automatically in CI pipelines
- **Container image scanning** blocks deployment of vulnerable images
- **Secret detection** prevents credentials from reaching repositories
- **Network isolation** is the default for all infrastructure templates
- **Managed identities** are configured instead of service account keys
- **Encryption at rest and in transit** is non-negotiable in all templates

Developers should never have to think about security basics — the platform handles them.

### Cost Controls
The platform enforces cost governance through:
- **Approved SKU lists** — only cost-appropriate resource sizes available
- **Resource quotas** — prevent runaway resource creation
- **Budget alerts** — notify teams before spending limits are reached
- **Auto-shutdown** — non-production resources stop outside business hours
- **Tagging enforcement** — every resource tagged for cost attribution
- **Right-sizing recommendations** — automated analysis of underutilized resources

### Compliance Evidence Automation
The platform generates compliance evidence automatically:
- **Policy compliance reports** — which resources comply, which don't
- **Change audit trails** — who changed what, when, and why
- **Access reviews** — automated reporting on who has access to what
- **Configuration drift detection** — alert when resources deviate from desired state
- **Deployment logs** — complete history of what was deployed and by whom

This eliminates the manual evidence collection that typically consumes weeks
during audit cycles. Auditors get real-time dashboards instead of spreadsheets.

### Governance Feedback Loop
Governance requirements evolve. The platform must adapt:
- Review policy effectiveness quarterly
- Remove policies that generate false positives without value
- Add policies when new compliance requirements emerge
- Gather feedback from developers on governance friction
- Balance security and compliance with developer productivity
