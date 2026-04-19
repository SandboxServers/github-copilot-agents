# Network Topology

### Hub-Spoke (Customer-Managed)
- Hub VNet with shared services (firewall, DNS, bastion, gateway) + spoke VNets peered to hub
- Spokes connect via VNet peering — **nontransitive** by default, traffic between spokes must route through hub NVA/firewall
- Hub contains: VPN/ExpressRoute gateway, Azure Firewall or NVA, Azure Bastion, DNS forwarders
- Gateway transit: enable "Allow gateway transit" on hub peering, "Use remote gateways" on spoke peering, "Allow forwarded traffic" on both
- UDRs on spoke subnets to route traffic through hub firewall (0.0.0.0/0 → firewall private IP)
- **Best for**: organizations needing full control over routing/firewall, existing NVA investments, complex UDR requirements
- **Limits**: VNet peering limit per VNet (500 default), manual route management, no built-in spoke-to-spoke transit

### Hub-Spoke with Virtual WAN (Microsoft-Managed)
- Microsoft-managed hub infrastructure — automated peering, routing, and transit
- Built-in spoke-to-spoke transit (full mesh) without manual UDRs
- Supports: S2S VPN, P2S VPN, ExpressRoute, inter-hub connectivity across regions
- Secured Virtual Hub = VWAN hub + Azure Firewall (managed by Firewall Manager)
- Routing intents: predefined routing for Internet traffic and Private traffic through security appliance
- Route maps: BGP-level control over advertised/received routes — filter, modify, permit/deny
- Hub extensions pattern for shared services (DNS, NVAs, Bastion) via dedicated spoke
- Scales to 20 Gbps aggregate throughput per hub
- **Best for**: large enterprises, multi-region deployments, SD-WAN integration, organizations wanting less operational overhead
- **Limits**: less UDR flexibility than customer-managed, some NVA limitations, cost per hub deployment hour + scale units + data processed

### When to Use Which
```
How many regions?
├─ 1 region, simple topology → Customer-managed hub-spoke
├─ 2+ regions needing transit → Virtual WAN (inter-hub connectivity)
├─ Complex custom routing/NVAs → Customer-managed (more UDR control)
├─ SD-WAN integration needed → Virtual WAN (partner integration)
├─ Want minimal ops overhead → Virtual WAN
└─ Need full route table control → Customer-managed
```
