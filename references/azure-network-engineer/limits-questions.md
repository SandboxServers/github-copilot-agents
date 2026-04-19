# Key Limits

| Resource | Limit |
|---|---|
| VNets per subscription per region | 1,000 |
| Subnets per VNet | 3,000 |
| VNet peerings per VNet | 500 |
| Private DNS zones per subscription | 1,000 |
| Record sets per private DNS zone | 25,000 |
| Virtual network links per private DNS zone | 1,000 |
| Auto-registration zones per VNet | 1 |
| Private endpoints per VNet | 1,000 (check current) |
| NSG rules per NSG | 1,000 |
| ExpressRoute route prefixes (private peering) | 4,000 (10,000 with Premium) |
| VPN Gateway S2S tunnels (VpnGw1) | 30 |
| VPN Gateway P2S connections | varies by SKU (up to 10,000) |
| Azure Firewall throughput | up to 30 Gbps (Premium) |

**Design rule**: always check current Azure subscription and service limits before designing — these change.
