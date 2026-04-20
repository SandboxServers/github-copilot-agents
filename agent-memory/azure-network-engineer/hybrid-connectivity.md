# Hybrid Connectivity

## VPN Gateway

Site-to-site (S2S) IPsec/IKEv2 tunnels over the public internet. Also supports Point-to-Site (P2S) for client VPN and VNet-to-VNet connections.

### SKUs and Throughput
- VpnGw1: ~650 Mbps, 30 S2S tunnels
- VpnGw2: ~1 Gbps, 30 S2S tunnels
- VpnGw3: ~1.25 Gbps, 30 S2S tunnels
- VpnGw4: ~5 Gbps, 100 S2S tunnels
- VpnGw5: ~10 Gbps, 100 S2S tunnels
- Generation 2 SKUs available for higher performance

### Features
- Active-active mode: dual tunnels for high availability
- BGP support: dynamic routing, automatic failover, transit routing
- P2S VPN: OpenVPN, IKEv2, SSTP protocols — supports Entra ID authentication
- Zone-redundant SKUs (AZ suffix) for availability zone resilience

### Gotchas
- Active-active + ASN 65515 required for Azure Route Server integration
- VPN gateways don't support BFD (Bidirectional Forwarding Detection)
- VPN gateways don't advertise default routes to other BGP peers

## ExpressRoute

Dedicated private connection via a connectivity provider. Traffic never traverses the public internet.

### Circuit Options
- Bandwidth: 50 Mbps to 10 Gbps (standard circuits via provider)
- ExpressRoute Direct: 10 Gbps or 100 Gbps dedicated port pairs, MACsec encryption
- Peering types: Private Peering (access VNets), Microsoft Peering (M365, Dynamics, Azure public services)
- Premium add-on: cross-region VNet connectivity, increased route limits (10,000 prefixes), Global Reach

### BGP and Routing
- BGP required for route exchange — up to 4,000 route prefixes (10,000 with Premium)
- BGP hold timer: 180 seconds (fixed on Microsoft side), keep-alive every 60 seconds
- Sessions disconnect if route prefix limit exceeded — restores when below limit
- Global Reach: connect two ExpressRoute circuits (site-to-site via Microsoft backbone)

### Failover
- ExpressRoute primary + S2S VPN backup (coexistence) — ExpressRoute preferred by default
- Dual ExpressRoute circuits in different peering locations for redundancy

## Comparison Table

| Criteria | VPN Gateway | ExpressRoute |
|---|---|---|
| Bandwidth | Up to 10 Gbps | Up to 100 Gbps (Direct) |
| Latency | Variable (internet path) | Consistent (dedicated connection) |
| Encryption | IPsec built-in | MACsec (Direct) or VPN overlay |
| Cost | Lower | Higher (circuit + provider fees) |
| Failover | Active-active VPN | ER circuits + VPN backup |
| SLA | 99.95% (active-active) | 99.95% (standard) |

## High Availability Patterns

### ExpressRoute + VPN Backup
- ExpressRoute as primary path — VPN gateway as automatic failover
- VPN activates if ExpressRoute circuit goes down
- Coexistence supported on same VNet with both gateways

### Dual ExpressRoute Circuits
- Two circuits from different peering locations (different provider edges)
- Protects against single peering location failure
- Both circuits can be active (load sharing) or active-passive

### Zone-Redundant Gateways
- Deploy gateway across availability zones (AZ-aware SKUs)
- Protects against single zone failure
- Available for both VPN and ExpressRoute gateways

## BGP Design

- Azure VPN Gateway default ASN: 65515
- ExpressRoute uses AS 12076 (Microsoft side) — customer provides their own ASN
- Azure Route Server: enables route exchange between gateways and NVAs without static routes
- Route preference: ExpressRoute routes preferred over VPN by default
- Use BGP weight and local-preference for path selection control
- ASN planning: document all ASNs across on-premises, Azure, and partner networks

## Forced Tunneling

- Redirect all internet-bound traffic (0.0.0.0/0) back to on-premises
- Methods: BGP (advertise 0.0.0.0/0 from on-premises) or set Default Site on VPN gateway
- **Critical**: if BGP session drops, internet traffic goes direct — add NSG outbound deny rules as safety net
- Alternative: route internet traffic through Azure Firewall instead of back to on-premises
- UDR method: 0.0.0.0/0 → NVA/Firewall in hub — ensure NVA has its own internet access

## Best Practices

- Always deploy redundant connectivity (dual circuits or ER + VPN)
- Use zone-redundant gateway SKUs in production
- Plan BGP ASNs upfront — document in IP/routing plan
- Monitor ExpressRoute circuit health and BGP session state
- Test failover scenarios regularly
- Use ExpressRoute for production workloads, VPN for dev/test or backup
