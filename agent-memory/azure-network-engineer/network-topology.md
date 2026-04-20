# Network Topology

## Hub-Spoke (Traditional, Customer-Managed)

Hub VNet contains shared services: Azure Firewall or NVA, VPN/ExpressRoute gateway, Azure Bastion, DNS forwarders. Spoke VNets peer to hub for connectivity.

### How It Works
- VNet peering connects each spoke to hub — peering is **non-transitive**
- Spoke A peered to Hub and Spoke B peered to Hub does NOT mean Spoke A can reach Spoke B
- Spoke-to-spoke traffic requires: UDR on spoke subnets (0.0.0.0/0 → hub firewall private IP) + IP forwarding enabled on NVA/Firewall
- Gateway transit: enable "Allow gateway transit" on hub peering, "Use remote gateways" on spoke, "Allow forwarded traffic" on both
- Each spoke is isolated by default — network segmentation built in

### Hub Components
- Azure Firewall or NVA for east-west and north-south traffic inspection
- VPN Gateway and/or ExpressRoute Gateway for hybrid connectivity
- Azure Bastion for secure VM access
- DNS forwarder VMs or Azure DNS Private Resolver
- Shared services subnet (Active Directory, monitoring, etc.)

### Strengths and Limits
- Full control over routing (custom UDRs), firewall rules, NVA placement
- Manual route management required — operational overhead scales with spoke count
- VNet peering limit: 500 per VNet (soft limit, can request increase)
- No built-in spoke-to-spoke transit — must engineer through hub

## Virtual WAN (Microsoft-Managed)

Microsoft-managed hub infrastructure with automated spoke connectivity and built-in transit routing.

### How It Works
- Spokes connect to VWAN hub — peering automated
- Built-in spoke-to-spoke transit: full mesh routing without manual UDRs
- Supports S2S VPN, P2S VPN, ExpressRoute, SD-WAN partner integration
- Inter-hub connectivity across regions via Microsoft global backbone
- Secured Virtual Hub = VWAN hub + Azure Firewall (managed via Firewall Manager)
- Routing intents: steer Internet traffic and Private traffic through security appliance
- Route maps: BGP-level control over advertised/received routes

### Strengths and Limits
- Less operational overhead — routing and transit handled by platform
- Scales to 20 Gbps aggregate throughput per hub
- SD-WAN integration with partners (Cisco, VMware, etc.)
- Less UDR flexibility than customer-managed hub-spoke
- Cost: hub deployment hour + scale units + data processed
- Some NVA deployment limitations compared to custom hub

## Comparison Table

| Criteria | Hub-Spoke | Virtual WAN |
|---|---|---|
| Management | Customer-managed | Microsoft-managed |
| Spoke-to-spoke | UDR + NVA required | Built-in transit |
| Max spokes | ~500 peerings per VNet | 500+ via VWAN hub |
| Cost model | Pay for NVAs/Firewall | Hub hours + data processing |
| Customization | Full UDR/route control | Less flexible |
| Multi-region | Manual inter-hub setup | Built-in inter-hub |
| SD-WAN support | Manual integration | Native partner integration |
| Operational overhead | Higher | Lower |

## IP Address Planning

- Plan CIDR blocks across ALL VNets and on-premises before deployment
- Never overlap address spaces — VNet peering fails if CIDRs overlap
- Recommended sizing: /16 for major VNets, /24 to /26 for subnets
- Reserve space for future growth (spokes, subnets, services)
- Document the IP plan — include VNet name, region, CIDR, subnet breakdown
- Use Azure IPAM or spreadsheet-based tracking
- Gateway subnet: /27 minimum (recommended /27)
- AzureFirewallSubnet: /26 minimum
- AzureBastionSubnet: /26 minimum

## Decision Guide

```
How many regions?
├─ 1 region, simple topology → Customer-managed hub-spoke
├─ 2+ regions needing transit → Virtual WAN (inter-hub connectivity)
├─ Complex custom routing/NVAs → Customer-managed (more UDR control)
├─ SD-WAN integration needed → Virtual WAN (partner integration)
├─ Minimal ops overhead wanted → Virtual WAN
└─ Full route table control needed → Customer-managed
```

## Best Practices

- Always deploy Azure Firewall or NVA in hub for east-west inspection
- Use UDRs on every spoke subnet to force traffic through hub
- Enable VNet peering monitoring and alerting
- Plan for growth — leave room in CIDR allocations
- Use naming conventions for VNets, subnets, and peerings
- Tag all networking resources for cost tracking and governance
