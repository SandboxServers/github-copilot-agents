## Management Group Hierarchy Design

Management groups provide a governance scope above subscriptions. Policies and RBAC assigned at a
management group are inherited by all child management groups and subscriptions. The ALZ hierarchy
organizes subscriptions by function and trust boundary.

### Root Management Group

The top-level management group sits directly under the Tenant Root Group. Apply tenant-wide
policies here that must apply to every subscription: allowed regions, required diagnostic settings,
Defender for Cloud enablement. Keep policies at this level minimal — overly restrictive root
policies create exemption sprawl.

Do not use the Tenant Root Group itself for policy assignment. Create an intermediate root MG
to preserve the ability to exclude subscriptions if needed.

### Platform Management Group

Contains subscriptions for shared platform services managed by the central team. No application
workloads belong here. Three child management groups partition platform concerns:

**Identity MG** — hosts the Identity subscription containing domain controllers (if hybrid
identity is required), Entra Connect or Cloud Sync servers, and related identity infrastructure.
Policies enforce strict access controls: no public IPs, no internet-facing endpoints, limited
admin access via PIM.

**Management MG** — hosts the Management subscription containing the central Log Analytics
workspace, Azure Automation accounts, Azure Monitor resources, Sentinel workspace, and Update
Management. All other subscriptions send diagnostic data here.

**Connectivity MG** — hosts the Connectivity subscription containing the hub VNet (or Virtual
WAN hub), Azure Firewall, VPN Gateway, ExpressRoute Gateway, Azure Bastion, and Private DNS
zones. The networking team owns this subscription. Policies enforce network security baselines.

### Landing Zones Management Group

Contains subscriptions where application teams deploy workloads. Two default child MGs provide
different policy profiles:

**Corp MG** — for internal workloads requiring private connectivity. Policies deny public IP
addresses on NICs, deny public endpoints on PaaS services, require private endpoints, and enforce
network security groups on all subnets. Spoke VNets peer to the hub for on-premises access.

**Online MG** — for internet-facing workloads. Policies allow controlled public endpoints (via
Application Gateway, Front Door, or App Service with access restrictions). Still requires NSGs
and diagnostic settings, but permits the public-facing patterns that Corp denies.

Organizations can add custom child MGs (e.g., SAP, Regulated) when workloads need distinct policy
sets that don't fit Corp or Online profiles.

### Sandbox Management Group

Contains subscriptions for experimentation, learning, and proof-of-concept work. Policies:
- No peering to hub (isolated from production networks)
- Relaxed compute and service policies (allow experimentation)
- Budget caps enforced (prevent runaway spend)
- Auto-delete or expiration policies where possible
- No access to production data or identity systems

Sandbox subscriptions give developers freedom to experiment without risking production resources
or violating compliance requirements in regulated management groups.

### Decommissioned Management Group

Subscriptions being retired are moved here. Policies:
- Deny all resource creation (no new deployments)
- Existing resources remain accessible for data extraction
- Scheduled for deletion after a retention period
- Activity logs continue flowing to central monitoring

Moving a subscription to Decommissioned is the first step in the retirement process, not the
last. Data migration and resource cleanup happen before the subscription is cancelled.