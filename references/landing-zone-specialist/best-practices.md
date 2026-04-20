## Landing Zone Best Practices

Proven practices for designing, deploying, and operating Azure landing zones.

> Source: Microsoft Cloud Adoption Framework, Azure Landing Zone Design Areas

---

### Start with CAF Reference Architecture, Customize from There

Do not design a management group hierarchy from scratch. Start with the ALZ reference architecture
(Platform → Connectivity/Identity/Management, Landing Zones → Corp/Online, Sandbox,
Decommissioned) and tailor it to your needs. Remove what you don't need rather than building up
from nothing. This ensures coverage of all 8 design areas and avoids missing critical components.

### Management Groups — Platform + Workload Separation

Structure management groups by archetype, not by org chart or environment. Platform management
group holds Connectivity, Identity, Management, and Security subscriptions. Landing Zones
management group holds Corp (hybrid-connected) and Online (internet-facing) workload archetypes.
Keep hierarchy flat — 3 to 4 levels maximum. Do not create separate management groups for
dev/test/prod environments; use separate subscriptions within the same management group instead.

### Subscription Vending Automation for Self-Service

Implement an automated subscription vending process so application teams can request and receive
properly configured subscriptions in minutes. Each vended subscription should arrive with correct
management group placement, baseline RBAC, network connectivity (VNet peering to hub), diagnostic
settings, and policy compliance. Use the ALZ subscription vending Bicep or Terraform modules as
accelerators. Treat vending as a platform product with an API or portal interface.

### Policy Strategy — Start Audit, Graduate to Deny

Deploy policies in Audit mode first to assess compliance posture without blocking deployments.
Communicate findings to workload teams and allow remediation time. Graduate to Deny only after
teams understand the requirements and have updated their deployments. Use DINE policies for
automated remediation of configuration drift (diagnostic settings, encryption, network rules).
Keep exemptions time-bound with documented remediation plans.

### Connectivity — Hub-Spoke or Virtual WAN, Decide Early

Choose network topology before deploying any workload. Hub-spoke is suitable for most enterprises
with fewer than 30 spoke VNets per hub. Virtual WAN suits organizations with many branch offices,
global presence, or large numbers of spokes. Both support Azure Firewall, ExpressRoute, and VPN.
Changing topology after workloads are deployed requires significant re-architecture. Decide once,
document the decision, and enforce via policy.

### Identity — Centralized Entra ID, PIM for Privileged Access

Establish identity baseline in the Identity subscription before onboarding workloads. Deploy
centralized Entra ID with Entra Connect (or cloud sync) for hybrid identity. Enable PIM for all
privileged roles — no standing admin access. Configure conditional access policies, enforce MFA,
and create break-glass accounts stored securely. Assign RBAC to groups, not individuals. Each
landing zone should have its own security groups — do not reuse groups across landing zones.

### Monitoring — Centralized Log Analytics in Management Subscription

Deploy a central Log Analytics workspace in the Management subscription under the Platform
management group. Use Azure Policy (DINE) to enforce diagnostic settings on all resources,
sending logs and metrics to the central workspace. Enable Microsoft Defender for Cloud continuous
export. Workload teams may have additional workspaces for application-specific telemetry, but
platform logs (activity logs, policy compliance, security alerts) must flow centrally.

### Security Baseline — Defender for Cloud on All Subscriptions

Enable Microsoft Defender for Cloud on every subscription, including sandbox and dev/test. Enable
Defender plans for the resource types in use (Servers, Storage, SQL, Key Vault, App Service, DNS,
Containers). Configure security contact email and notification settings. Export Defender alerts
and recommendations to the central Log Analytics workspace. Use Secure Score as a governance KPI.

### Tagging — Enforce via Policy, Not Trust

Define a minimum tagging taxonomy (CostCenter, Owner, Environment, Application, Criticality) and
enforce required tags via Azure Policy with deny effect. Do not rely on teams voluntarily tagging
resources. Use Modify policies to inherit tags from resource groups or subscriptions where
appropriate. Tags drive cost allocation, automation, and operational visibility — inconsistent
tagging undermines all three.

### Regular Compliance Reviews with Remediation Plans

Schedule monthly reviews of Azure Policy compliance dashboard, Defender for Cloud Secure Score,
and landing zone health metrics. For each non-compliant finding, create a time-bound remediation
plan with a responsible owner. Track remediation completion rate as a platform KPI. Treat the
landing zone as a product — maintain a backlog, ship incremental improvements, and gather feedback
from workload teams through regular architecture review boards.

---

**See also:** [anti-patterns.md](anti-patterns.md) for common mistakes,
[gotchas.md](gotchas.md) for operational surprises.
