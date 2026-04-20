# Network Limits and Common Questions

## Key Limits

| Resource | Limit |
|---|---|
| VNets per subscription per region | 1,000 |
| Subnets per VNet | 3,000 |
| VNet peerings per VNet | 500 |
| Private endpoints per VNet | 1,000 |
| NSG rules per NSG | 1,000 |
| UDRs per route table | 400 |
| Private DNS zones per subscription | 1,000 |
| Record sets per private DNS zone | 25,000 |
| Virtual network links per private DNS zone | 1,000 |
| Auto-registration zones per VNet | 1 |
| ExpressRoute route prefixes (private peering) | 4,000 (10,000 with Premium) |
| VPN Gateway S2S tunnels (VpnGw1) | 30 |
| VPN Gateway P2S connections | varies by SKU (up to 10,000) |
| Azure Firewall throughput | up to 30 Gbps (Premium) |

**Design rule**: always check current Azure subscription and service limits before designing — limits change over time.

## Common Questions

### Can spokes communicate in hub-spoke?
Not by default. VNet peering is non-transitive. Spoke-to-spoke requires:
- UDR on spoke subnets pointing 0.0.0.0/0 to hub NVA/Firewall private IP
- IP forwarding enabled on the NVA/Firewall
- Firewall rules allowing the spoke-to-spoke traffic

### Is VNet peering transitive?
No. If VNet A is peered with VNet B, and VNet B is peered with VNet C, VNet A cannot reach VNet C through B. You must either:
- Peer A directly with C
- Route through a hub NVA/Firewall (hub-spoke pattern)

### Private endpoint vs service endpoint?
- **Private endpoint**: private IP in your VNet, full isolation, works from on-premises, per-instance, costs money
- **Service endpoint**: backbone routing with public IP, VNet-only access, per-service-type, free
- Use private endpoints for production; service endpoints for budget-constrained or simple scenarios

### Is ExpressRoute encrypted?
Not by default. The dedicated circuit is private but not encrypted. Options:
- MACsec encryption (ExpressRoute Direct only)
- IPsec VPN overlay on top of ExpressRoute
- Application-level encryption (TLS)

### How to force tunnel all internet traffic?
- UDR: 0.0.0.0/0 → NVA/Firewall private IP in hub on all spoke subnets
- Ensure the NVA/Firewall has its own internet access (don't create a routing loop)
- Alternative: BGP advertise 0.0.0.0/0 from on-premises for VPN/ExpressRoute scenarios

### Can I use the same subnet for VNet integration and private endpoints?
No. VNet integration subnets are delegated to `Microsoft.Web/serverFarms` and cannot host private endpoints. Use separate subnets.

### What happens if NSG flow logs are enabled on both subnet and NIC NSGs?
Both log independently, which creates duplication. Prefer VNet flow logs (newer feature) over NSG flow logs for simpler scope and less duplication.
