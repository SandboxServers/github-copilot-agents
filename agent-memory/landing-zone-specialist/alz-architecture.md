## Azure Landing Zone Conceptual Architecture

The Azure Landing Zone (ALZ) is a prescriptive architectural approach that provides a scalable,
secure foundation for Azure workloads. It defines the management group hierarchy, policy baseline,
networking topology, and identity model that all subscriptions inherit.

### Management Group Hierarchy

```
Tenant Root Group
└─ Root Management Group (organization-level policies)
   ├─ Platform
   │  ├─ Identity        → Domain controllers, Entra Connect
   │  ├─ Management      → Log Analytics, Automation, Sentinel
   │  └─ Connectivity    → Hub VNet, VPN/ExpressRoute, Firewall, DNS
   ├─ Landing Zones
   │  ├─ Corp            → Internal workloads, private connectivity only
   │  └─ Online          → Internet-facing workloads, public endpoints allowed
   ├─ Decommissioned     → Subscriptions being retired, deny-all policies
   └─ Sandbox            → Experimentation, no production connectivity
```

Platform subscriptions are managed by the central platform team. Landing zone subscriptions are
delegated to application teams who operate within guardrails set by policy inheritance.

### Eight Design Areas

| # | Design Area | Scope |
|---|------------|-------|
| 1 | **Billing and Tenant** | EA/MCA enrollment structure, tenant topology, billing scopes |
| 2 | **Identity and Access** | Entra ID configuration, RBAC model, PIM, conditional access |
| 3 | **Network Topology** | Hub-spoke or VWAN, DNS, ExpressRoute/VPN, private endpoints |
| 4 | **Resource Organization** | Management group hierarchy, subscription strategy, naming |
| 5 | **Governance** | Azure Policy, tagging, cost management, compliance reporting |
| 6 | **Security** | Defender for Cloud, Sentinel, Key Vault, DDoS Protection |
| 7 | **Management** | Monitoring, patching, backup, disaster recovery, alerting |
| 8 | **Platform Automation** | IaC tooling (Bicep/Terraform), CI/CD pipelines, GitOps |

Each design area requires decisions that range from easy-to-change-later (monitoring tooling) to
extremely-hard-to-change-later (network topology, IP addressing, identity architecture).

### Deployment Approaches

**Start small and expand** — deploy the management group hierarchy with policies in audit mode,
stand up the Connectivity and Management subscriptions, and onboard workloads incrementally. Best
for organizations new to Azure or with limited platform team capacity. Allows learning and
iteration before committing to enterprise-scale decisions.

**Enterprise-scale deployment** — deploy the full ALZ reference architecture using an accelerator
(Portal, Bicep, or Terraform). Includes all management groups, policy assignments, hub networking,
and platform subscriptions in a single deployment. Best for organizations with clear requirements,
experienced platform teams, and executive sponsorship.

### Accelerator Options

| Accelerator | Tooling | Best For |
|-------------|---------|----------|
| Portal Accelerator | Azure Portal wizard | Quick greenfield setup, demos |
| ALZ Bicep | Bicep modules (`Azure/ALZ-Bicep`) | Organizations standardized on Bicep |
| ALZ Terraform | Terraform module (`Azure/terraform-azurerm-caf-enterprise-scale`) | Organizations standardized on Terraform |
| Sovereign Landing Zone | Bicep | Government and sovereign cloud requirements |

Regardless of accelerator choice, the conceptual architecture and design principles remain the
same. The accelerator is the deployment mechanism, not the architecture itself.

### Design Principles

- **Subscription democratization** — subscriptions as units of management, not shared containers
- **Policy-driven governance** — guardrails enforced through Azure Policy, not manual review
- **Single control plane** — Azure Resource Manager as the consistent management layer
- **Application-centric migration** — align subscriptions to applications, not infrastructure tiers
- **Architecture aligned with Azure platform roadmap** — use native services over custom solutions

The ALZ architecture is a reference, not a rigid template. Adapt the hierarchy and policies to
organizational needs while preserving the structural principles that make the pattern work.