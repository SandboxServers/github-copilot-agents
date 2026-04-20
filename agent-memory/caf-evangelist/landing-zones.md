## Landing Zones — Ready Phase

### What Is a Landing Zone

A landing zone is a pre-configured Azure environment that provides the foundation for
workload deployment. It implements networking, identity, governance, security, and
management capabilities before any application workloads arrive.

### Azure Landing Zone (ALZ) Conceptual Architecture

The ALZ reference architecture covers eight design areas:

1. **Billing and Entra tenant** — tenant structure, billing scopes
2. **Identity and access** — Entra ID, RBAC, PIM, conditional access
3. **Management groups and subscriptions** — hierarchy, subscription vending
4. **Network topology** — hub-spoke or Virtual WAN, DNS, connectivity
5. **Security** — Defender for Cloud, Sentinel, Key Vault
6. **Management** — monitoring, patching, backup, disaster recovery
7. **Governance** — Azure Policy, tagging, cost management
8. **Platform automation** — IaC, CI/CD, deployment pipelines

### Deployment Approaches

**Start Small and Expand** — begin with a minimal landing zone and iterate:
- Suitable for smaller organizations or teams new to Azure
- Deploy foundational policies and networking first
- Add design areas incrementally as needs emerge
- Lower upfront investment, faster initial deployment

**Enterprise-Scale** — deploy the full ALZ reference architecture:
- Suitable for large organizations with complex requirements
- All eight design areas addressed from day one
- Higher upfront investment, comprehensive foundation
- Recommended when compliance or scale demands it

### Management Group Hierarchy

```
Tenant Root Group
└─ Organization Root
   ├─ Platform
   │  ├─ Management    (Log Analytics, Automation, Sentinel)
   │  ├─ Connectivity  (Hub networking, DNS, Firewall)
   │  └─ Identity      (Domain controllers, Entra Connect)
   ├─ Landing Zones
   │  ├─ Corp          (internal workloads, private connectivity)
   │  └─ Online        (internet-facing workloads)
   ├─ Sandbox          (dev/test, no prod connectivity)
   └─ Decommissioned   (retired subscriptions, restrictive policies)
```

### Platform vs Application Landing Zones

| Type | Managed By | Purpose |
|------|-----------|---------|
| **Platform Landing Zone** | Central platform team | Shared services: networking, identity, monitoring, security |
| **Application Landing Zone** | App teams within guardrails | Workload hosting with inherited governance and connectivity |

Platform landing zones provide the shared infrastructure that application landing zones
consume. App teams get a subscription with networking, policies, and RBAC pre-configured.

### Hub-Spoke vs Virtual WAN

| Aspect | Hub-Spoke | Virtual WAN |
|--------|----------|-------------|
| Complexity | Medium — manual peering, UDRs, NVAs | Lower — Microsoft-managed routing |
| Control | Full control over routing and appliances | Less granular control |
| Scale | Good for moderate topology | Better for large-scale branch connectivity |
| Cost | NVA licensing, hub VM costs | VWAN hub hourly + data processing |
| Team skill | Requires network engineering depth | Simpler operations |

### Policy-Driven Governance

Landing zones enforce standards through Azure Policy:
- **Deny** policies prevent non-compliant resource creation
- **Audit** policies flag violations without blocking
- **DeployIfNotExists** policies auto-remediate (e.g., enable diagnostics)
- **Modify** policies adjust resource properties at deployment time

Policies are assigned at management group level and inherited by all child subscriptions.

### Subscription Vending

Automate new subscription provisioning:
- Self-service portal or IaC pipeline requests new subscriptions
- Subscriptions placed in correct management group automatically
- Networking, RBAC, policies, and monitoring configured at creation
- Reduces provisioning time from weeks to minutes
- Eliminates manual configuration drift
