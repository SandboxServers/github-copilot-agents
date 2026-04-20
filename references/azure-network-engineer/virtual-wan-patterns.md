# Azure Virtual WAN Patterns

## When to Choose Virtual WAN over Hub-Spoke

| Factor | Traditional Hub-Spoke | Virtual WAN |
|--------|----------------------|-------------|
| Spoke count | < 30 spokes | 30+ spokes or rapid growth |
| Transitive routing | Manual (UDR + NVA) | Built-in — any-to-any by default |
| Multi-region | Manual inter-hub peering | Automated hub-to-hub backbone |
| Branch connectivity | Customer-managed VPN config | Automated branch-to-hub (SD-WAN partners) |
| Operational overhead | Full control, more work | Microsoft-managed routing, less control |
| Cost | Pay per resource | Hub fee + data processing — can be higher for small deployments |

**Rule of thumb:** If you need full control over routing and NVA placement, use traditional hub-spoke. If you need scale, multi-region transit, and automated branch onboarding, use Virtual WAN.

## Virtual WAN Architecture

### Components
- **Virtual WAN resource** — top-level container, one per organization or region group
- **Virtual Hub** — Microsoft-managed VNet in a region; hosts gateway instances and routing
- **Hub VNet connections** — connect spoke VNets to the hub (replaces manual peering)
- **VPN Site** — represents a branch location for S2S VPN
- **ExpressRoute circuit** — connects on-premises via private peering to the hub
- **P2S VPN Gateway** — remote user VPN built into the hub

### Key Capabilities
- **Any-to-any connectivity** — spoke-to-spoke, branch-to-spoke, branch-to-branch all routed through the hub by default
- **Hub-to-hub transit** — traffic between regions traverses the Microsoft global backbone automatically
- **Routing intent** — send all private and/or internet traffic through a security solution (Azure Firewall, NVA) in the hub
- **BGP peering with NVAs** — inject custom routes from NVAs deployed in spoke VNets into the hub route table

## Routing Intent and Policies

Routing intent simplifies the "force all traffic through firewall" pattern:

- **Private traffic policy** — routes all spoke-to-spoke and branch-to-spoke traffic through the security solution in the hub
- **Internet traffic policy** — routes all internet-bound traffic through the security solution
- When routing intent is enabled, the hub advertises default routes (0.0.0.0/0 and RFC1918 prefixes) to all connected spokes and branches

### Configuration
1. Deploy Azure Firewall or supported NVA in the Virtual Hub
2. Enable Routing Intent → select private traffic, internet traffic, or both
3. All connected VNets and branches automatically receive the routes — no manual UDRs needed

## Common Patterns

### Secured Virtual Hub (vWAN + Azure Firewall)
- Azure Firewall deployed inside the Virtual Hub
- Routing intent enabled for both private and internet traffic
- All spoke and branch traffic inspected centrally
- Simplest way to achieve full inspection with Virtual WAN

### Virtual WAN + Third-Party NVA
- NVA deployed in a spoke VNet (not in the hub)
- BGP peering between NVA and Virtual Hub to advertise routes
- Use when the organization requires a specific vendor firewall (Palo Alto, Fortinet, etc.)
- More complex than Azure Firewall in-hub but supports vendor-specific features

### Multi-Region with Routing Intent
- One Virtual Hub per region, both with Azure Firewall + routing intent
- Inter-hub traffic traverses Microsoft backbone and is inspected at each hub
- On-premises connected via ExpressRoute to the nearest hub; cross-region traffic flows hub-to-hub

## Virtual WAN Gotchas

- **No custom route tables** when routing intent is enabled — Microsoft manages all routing
- **Spoke VNets cannot peer with each other** when connected to Virtual WAN; all traffic goes through the hub
- **NVA in the hub** is limited to specific partners (Check Point, Fortinet, Barracuda); arbitrary NVAs require spoke deployment + BGP
- **Cost** — Virtual Hub has a per-hour fee plus data processing charges. Small deployments may cost more than traditional hub-spoke
- **Less control** over routing — you trade fine-grained UDR control for automated management
- **Migration from traditional hub-spoke** requires re-creating peerings as hub VNet connections; plan for connectivity downtime

---

*Sources: [Virtual WAN overview](https://learn.microsoft.com/azure/virtual-wan/virtual-wan-about), [Routing intent](https://learn.microsoft.com/azure/virtual-wan/how-to-routing-policies), [Virtual WAN architecture](https://learn.microsoft.com/azure/architecture/networking/architecture/hub-spoke-vwan-architecture)*
