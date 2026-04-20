## Landing Zone Anti-Patterns

Broad anti-patterns observed in Azure landing zone design, deployment, and operations. For
ALZ-specific structural anti-patterns, see [alz-anti-patterns.md](alz-anti-patterns.md).

> Source: Microsoft Cloud Adoption Framework — Landing Zone Anti-Patterns

---

### Deploying Workloads Before the Landing Zone Is Ready

Teams deploy production workloads into unstructured subscriptions while the platform team is still
building governance foundations. This creates ungoverned resources that must be retroactively
brought into compliance — far harder than getting it right from the start. Establish at minimum
management group hierarchy, baseline policies, and connectivity before onboarding workloads.

### Too-Deep Management Group Hierarchy

Exceeding 3–4 levels of management groups adds management overhead and complexity. Policy
debugging becomes difficult because inheritance chains are long and hard to trace. CAF recommends
no more than 3–4 levels. Use management groups for archetype-based policy assignment (Corp,
Online, Sandbox), not to mirror org charts or SDLC environments.

### No Subscription Vending — Manual Subscription Creation

Manual subscription provisioning (ticket queues, email chains, weeks of waiting) leads to shared
subscriptions, inconsistent configuration, and teams bypassing the process entirely. Without
automated vending, shadow IT proliferates — teams create subscriptions via personal billing
accounts in unmanaged tenants. Implement self-service subscription vending with IaC.

### Policy Exemptions as Permanent Workarounds

Granting policy exemptions for "temporary" issues that never get resolved. Exemptions accumulate,
governance erodes, and compliance reports become meaningless. Treat every exemption as a
time-boxed exception with a remediation deadline. Review exemptions monthly and revoke expired
ones. Track exemption count as a platform health metric.

### Skipping Network Topology Decisions

Deferring the hub-spoke vs. Virtual WAN decision until "later" leads to ad-hoc networking:
direct peering between workload VNets, no central firewall, no DNS strategy. Retrofitting a
hub-spoke or vWAN topology after dozens of VNets exist is painful. Decide on network topology
during landing zone design, before any workload deployment.

### Not Separating Platform from Workload Subscriptions

Mixing platform resources (firewalls, DNS, domain controllers, Log Analytics) with application
workloads in the same subscriptions. This violates blast-radius isolation, makes RBAC chaotic,
and couples platform lifecycle to workload lifecycle. Use dedicated Platform subscriptions
(Connectivity, Identity, Management) under the Platform management group.

### Single Subscription for Everything

Putting all workloads into one subscription eliminates isolation, makes RBAC unmanageable,
prevents meaningful cost allocation, and hits subscription-level limits (800 resource groups,
quota limits per resource type). Subscriptions are free — use them as isolation and scale-unit
boundaries. One subscription per workload per environment is the default pattern.

### Not Planning for Brownfield Adoption

Assuming a greenfield deployment when the organization already has Azure workloads. Moving
existing subscriptions into the new management group hierarchy changes inherited policies and
RBAC immediately. DINE policies show existing resources as non-compliant but do not auto-
remediate them. Plan subscription migration carefully: inventory resources, assess policy
impact, remediate in waves, and validate after each move.

### Custom Governance Instead of Azure Policy

Building custom scripts, Azure Functions, or third-party tools for governance tasks that Azure
Policy handles natively (tagging enforcement, allowed regions, required SKUs, diagnostic
settings). Custom governance adds maintenance burden, lacks the compliance dashboard, and drifts
silently. Use Azure Policy as the primary governance mechanism — extend with custom solutions
only for gaps Policy cannot cover.

### Not Implementing Identity Baseline Before Workloads

Deploying workloads without first establishing centralized Entra ID integration, PIM for
privileged roles, conditional access policies, and break-glass accounts. Workloads then ship
with local admin accounts, no MFA, and inconsistent RBAC. Identity is the new perimeter —
establish identity baseline in the Identity subscription before onboarding workload teams.

### Landing Zone as "Project" Not "Product"

Treating the landing zone as a one-time project that ends after initial deployment. No ongoing
maintenance, no policy updates, no module upgrades, no feedback loop from workload teams. Landing
zones are a product with continuous customers (application teams). Assign a platform team that
owns the lifecycle: backlog, releases, SLAs, and regular architecture reviews.

### Ignoring CAF Reference Architecture Patterns

Designing landing zone hierarchy from scratch instead of starting with the CAF reference
architecture and customizing. This often produces missing management groups (no Decommissioned,
no Sandbox), inconsistent policy assignment, and gaps in design areas (security, governance,
identity). Start from the reference architecture and tailor — don't reinvent.

---

**See also:** [alz-anti-patterns.md](alz-anti-patterns.md) for ALZ-specific structural anti-patterns.
