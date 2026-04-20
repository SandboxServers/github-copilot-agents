## Network Topology

### Hub-Spoke in Landing Zone Context

The hub VNet lives in the **Connectivity subscription** under the Platform management group. It
hosts shared networking services: Azure Firewall (or third-party NVA), VPN Gateway, ExpressRoute
Gateway, and Azure Bastion. The platform networking team owns and operates the hub.

Spoke VNets live in **landing zone subscriptions** (one per workload/environment). Each spoke is
peered to the hub, giving workloads access to on-premises connectivity, shared services, and
internet egress through the hub firewall.

### Virtual WAN Alternative

Virtual WAN provides a Microsoft-managed hub with built-in any-to-any transit routing, automatic
hub-to-hub connectivity for multi-region, and native SD-WAN partner integration. Choose VWAN when
you have more than 30 branch sites, need VPN-to-ExpressRoute transitive routing, or want lower
operational overhead at the cost of less customization.

### Hub-Spoke vs VWAN Summary

| Factor | Hub-Spoke | Virtual WAN |
|--------|-----------|-------------|
| Hub management | Customer-managed VNet | Microsoft-managed service |
| Spoke-to-spoke | Through firewall/NVA | Built-in any-to-any |
| Multi-region | Manual hub peering | Automatic hub-to-hub |
| NVA flexibility | Any NVA | Limited to supported NVAs |
| UDR control | Full | Limited (improving) |
| Branch scale | ≤30 tunnels practical | 1000s of branches |
| Operational overhead | Higher | Lower |

### Peering from Landing Zone to Hub

Every landing zone spoke VNet peers to the hub. Peering configuration:
- **Allow forwarded traffic** — enabled on both sides
- **Allow gateway transit** — enabled on hub side (shares VPN/ER gateway)
- **Use remote gateways** — enabled on spoke side
- Peering is non-transitive: spokes cannot talk to each other without going through the hub

### Forced Tunneling Through Hub Firewall

User-defined routes (UDRs) on spoke subnets send all traffic (0.0.0.0/0) to the hub firewall's
private IP. The firewall inspects, logs, and filters traffic before forwarding to the internet or
on-premises. This provides centralized egress control, threat inspection, and logging.

### Private DNS Zones in Hub

Azure Private DNS zones are hosted in the **Connectivity subscription** and linked to all spoke
VNets. This centralizes DNS management for private endpoints. When a workload creates a private
endpoint, the DNS record is created in the central zone, and all spokes can resolve it.

Recommended zones include: `privatelink.blob.core.windows.net`, `privatelink.database.windows.net`,
`privatelink.vaultcore.azure.net`, and dozens more for each PaaS service using private endpoints.

### IP Address Planning

Plan IP address space across **all subscriptions and on-premises** before deploying anything:
- Assign non-overlapping CIDR blocks to each spoke VNet
- Reserve space for hub subnets (GatewaySubnet, AzureFirewallSubnet, AzureBastionSubnet)
- Account for on-premises ranges to avoid conflicts
- Plan for growth — expanding address space later requires re-peering or adding ranges
- Use a central IPAM tool or spreadsheet as the source of truth
- Typical spoke: /22 to /24 depending on workload size
- Typical hub: /22 minimum (firewall subnet needs /26, gateway needs /27)

Overlapping IP ranges between Azure and on-premises cause routing failures that are extremely
painful to remediate after workloads are deployed. Get IP planning right before the first spoke.
