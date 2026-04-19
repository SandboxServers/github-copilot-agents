# Hybrid Connectivity

### ExpressRoute
- Private, dedicated connection via connectivity provider — not over public internet
- Bandwidths: 50 Mbps to 100 Gbps depending on provider and circuit SKU
- Peering types: Private Peering (VNets), Microsoft Peering (M365, Dynamics, Azure public services)
- BGP for dynamic route exchange — up to 4,000 route prefixes (10,000 with Premium)
- Premium add-on: increased route limits, cross-region VNet connectivity, global reach
- BGP hold timer: 180 seconds (fixed on Microsoft side), keep-alive every 60 seconds
- **Gotcha**: BGP sessions disconnect if route limit exceeded — session restores when below limit
- ExpressRoute Direct: dedicated 10/100 Gbps port pair, MACsec encryption
- **Failover**: ExpressRoute + S2S VPN coexistence — ExpressRoute preferred by default, VPN as backup

### VPN Gateway
- IPsec/IKE tunnels over public internet — S2S, P2S, VNet-to-VNet
- SKUs: VpnGw1-5 (generation 1/2) — throughput from 650 Mbps to 10 Gbps
- Active-active mode: dual tunnels for HA — required for BGP with Azure Route Server
- BGP support: dynamic routing, automatic failover, transit routing
- **Gotcha**: active-active + ASN 65515 required for Route Server integration
- **Gotcha**: VPN gateways don't support BFD (Bidirectional Forwarding Detection)
- **Gotcha**: VPN gateways don't advertise default routes to other BGP peers

### Virtual WAN Connectivity
- Aggregates VPN, ExpressRoute, P2S, SD-WAN into single hub
- Inter-hub connectivity across regions via Microsoft global backbone
- Routing intents replace manual UDRs for traffic steering through security appliance
- Cost: deployment hours + scale units + connection units + data processed

### Forced Tunneling
- Redirect all internet-bound traffic (0.0.0.0/0) back to on-premises via VPN/ExpressRoute
- Methods: BGP (advertise 0.0.0.0/0 from on-premises), Default Site on VPN gateway
- **Critical**: if BGP session drops, internet traffic goes direct — add NSG outbound deny rules as safety net
- Alternative: route internet traffic through Azure Firewall instead of back to on-premises

### BGP Design
- Azure VPN Gateway default ASN: 65515
- ExpressRoute uses AS 12076 (Microsoft) — customer provides their own ASN
- Azure Route Server: enables route exchange between gateways and NVAs without manual static routes
- Route preference: ExpressRoute routes preferred over VPN by default
- Symmetric routing: set local preference > 100 on ExpressRoute-learned routes on-premises
