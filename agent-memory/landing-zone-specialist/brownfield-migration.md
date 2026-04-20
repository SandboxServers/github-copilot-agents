## Brownfield Migration to ALZ

### Assessment Phase

Before migrating an existing Azure environment to the ALZ architecture, inventory the current
state across five dimensions:

**Current subscriptions** — how many exist, what is deployed in each, who owns them, and what
management group (if any) are they in. Identify shared subscriptions with multiple unrelated
workloads that need to be separated.

**Management group structure** — does one exist or are subscriptions under the Tenant Root Group.
Document any existing policies and RBAC assignments at each level.

**Policy state** — what Azure Policies are currently assigned, at what scope, and in what effect
mode. Identify gaps (no policies at all) and conflicts (policies that contradict ALZ defaults).

**Networking** — map all VNets, peerings, public IPs, DNS configurations, and on-premises
connectivity (VPN/ExpressRoute). Identify overlapping IP address ranges that must be resolved.

**Monitoring** — where do logs go today, is Defender for Cloud enabled, are diagnostic settings
configured, and is there a central Log Analytics workspace.

### Phased Migration Approach

**Phase 1 — Deploy ALZ structure alongside existing environment.**
Create the management group hierarchy (Platform, Landing Zones, Sandbox, Decommissioned) without
moving any existing subscriptions. Deploy the Connectivity, Management, and Identity subscriptions
as new platform subscriptions. This is non-disruptive to existing workloads.

**Phase 2 — Move subscriptions to correct management groups.**
Start with non-production subscriptions. Moving a subscription to a new management group changes
which policies it inherits. Deploy all ALZ policies in **Audit mode** before moving subscriptions
so the move does not break existing deployments.

**Phase 3 — Remediate policy compliance.**
Review the compliance dashboard after subscription moves. Work with application teams to fix
non-compliant resources: add missing tags, enable encryption, configure diagnostic settings,
remove public endpoints. Use remediation tasks for DINE policies.

**Phase 4 — Migrate networking to hub-spoke.**
This is the most disruptive phase. Deploy the hub VNet with firewall in the Connectivity
subscription. Re-peer existing spoke VNets to the new hub. Update UDRs for forced tunneling.
Migrate DNS to central Private DNS zones. Plan maintenance windows for connectivity cutover.

**Phase 5 — Consolidate monitoring.**
Route all diagnostic settings to the central Log Analytics workspace in the Management
subscription. Enable Defender for Cloud across all subscriptions. Deploy Sentinel for security
monitoring. Decommission redundant monitoring solutions.

**Phase 6 — Enforce and operationalize.**
Promote policies from Audit to Deny/DINE after remediation is complete. Enable subscription
vending for new workload requests. Document operational procedures and train teams.

### Risk Management

**Policy impact assessment** — before assigning any policy in Deny mode, run it in Audit mode
for at least 4 weeks. Review the compliance report to understand exactly which existing resources
would be affected. Communicate findings to owning teams before enforcement.

**Subscription move risks** — moving a subscription to a new management group immediately changes
inherited policies and RBAC. Test with a non-critical subscription first. Have a rollback plan
(move back to original MG if issues arise).

**Network migration risks** — changing peering and routing affects connectivity for running
workloads. Plan migrations during maintenance windows. Test connectivity after each change.
Keep old peerings active until new routing is confirmed working.

### Brownfield Red Flags

- Subscriptions sitting directly under Tenant Root Group
- Service principals with Owner role on entire subscriptions
- No Azure Policy assignments anywhere
- Overlapping IP address spaces between VNets or with on-premises
- Public IPs directly on VM NICs
- No diagnostic settings (zero operational visibility)
- Classic (ASM) resources still running