# Azure Networking Best Practices

## IP Address Planning

- **Document all address ranges** in a central IPAM tool or spreadsheet. Track VNet, subnet, on-premises, and reserved ranges.
- **Use /16 for VNets** as the default starting point. This gives 65,536 addresses and room for many subnets.
- **Use /24 for subnets** unless the workload specifically needs more or fewer IPs. Remember Azure reserves 5 IPs per subnet.
- **Leave room for growth.** Don't pack subnets tightly. Future services (private endpoints, App Service integration, AKS node pools) consume IPs faster than expected.
- **Never overlap** address spaces across VNets, on-premises networks, or environments. Overlapping CIDRs break peering and VPN connectivity.

## Hub-Spoke Topology

- **Use hub-spoke as the default architecture.** Hub VNet hosts shared services (firewall, gateways, DNS, Bastion). Spokes host workloads.
- **One hub per region.** Multi-region deployments get one hub per region with inter-hub peering or Global VNet Peering.
- **Enable gateway transit** on the hub peering and "Use remote gateways" on spoke peerings so spokes share the hub's VPN/ExpressRoute gateway.
- **Route spoke-to-spoke through the hub firewall** using UDRs (0.0.0.0/0 → firewall private IP) on spoke subnets. This enforces inspection and logging.
- **Consider Azure Virtual WAN** for large-scale deployments (50+ spokes, multi-region) where transitive routing and automated spoke management justify the cost.

## Private Connectivity

- **Use private endpoints for all supported PaaS services.** Storage, SQL, Key Vault, Cosmos DB, Event Hubs, Service Bus — if it supports private endpoints, use them.
- **Disable public network access** on PaaS resources after enabling private endpoints.
- **Create Private DNS Zones** for each service type (e.g., `privatelink.blob.core.windows.net`) and link them to the hub VNet.
- **Centralize DNS resolution** through the hub using Azure DNS Private Resolver or conditional forwarders on hub DNS servers. Spokes should use the hub's DNS.
- **Use service endpoints** as a simpler alternative when private endpoints aren't required and you just need to keep traffic on the Azure backbone.

## Network Security Groups

- **NSG on every subnet** — no exceptions. Even "internal" subnets should have an NSG with explicit rules.
- **Start with deny-all inbound**, then add specific allow rules. Default Azure rules allow intra-VNet and load balancer traffic.
- **Use service tags** (`AzureCloud`, `Storage`, `Sql`, `AzureMonitor`, `Internet`) instead of hardcoded IP addresses. Service tags are maintained by Microsoft.
- **Use Application Security Groups (ASGs)** to group VMs by role (web, app, db) and write rules like "web ASG → app ASG on port 8080" instead of managing individual IPs.
- **Tag every NSG** with `Purpose`, `Owner`, `Environment`. Unnamed, untagged NSGs are impossible to manage at scale.
- **Review NSG rules quarterly.** Remove stale allow rules. Check for overly permissive rules (`*` source or `*` port).

## Firewalling & Inspection

- **Use Azure Firewall or a third-party NVA** for centralized east-west (spoke-to-spoke) and north-south (internet) traffic inspection.
- **Deploy Azure Firewall across Availability Zones** for 99.99% SLA.
- **Use Azure Firewall Premium** for TLS inspection, IDPS, and URL filtering when compliance requires deep packet inspection.
- **Use Firewall Policy** (not classic rules) for hierarchical, reusable rule sets across multiple firewalls.

## Hybrid Connectivity

- **ExpressRoute + VPN failover.** Use S2S VPN as a backup path with lower BGP weight so failover is automatic.
- **Use BGP on all VPN connections** for dynamic route exchange. Avoid static routes which require manual updates.
- **Enable BFD (Bidirectional Forwarding Detection)** on ExpressRoute private peering for faster failover detection.
- **Monitor ExpressRoute circuit utilization** and set alerts at 70% bandwidth to prevent saturation.

## Monitoring & Operations

- **Enable Network Watcher in every region** where you have resources. It's required for flow logs, packet capture, topology views, and connection troubleshoot.
- **Enable VNet flow logs** (preferred over NSG flow logs) and send to Log Analytics. Use Traffic Analytics for visualization and anomaly detection.
- **Use Connection Monitor** for end-to-end connectivity monitoring between VMs, VNets, and on-premises endpoints.
- **Check effective routes and effective security rules** during troubleshooting — not just the configured route tables and NSGs.
- **Enable diagnostic settings** on Azure Firewall, Application Gateway, VPN Gateway, and Load Balancer. Send logs to Log Analytics.

## DDoS & Edge Protection

- **Enable DDoS Protection Standard** on VNets with public-facing resources. It provides enhanced mitigation, telemetry, cost protection, and SLA guarantees.
- **Use Azure Front Door** for global HTTP/HTTPS workloads — built-in WAF, DDoS protection, SSL offload, and global load balancing.
- **Rate-limit and geo-filter** at the edge (Front Door or Application Gateway WAF) before traffic reaches backend services.

## Infrastructure as Code

- **Define all networking in Bicep or Terraform.** NSGs, route tables, peerings, firewall rules — all should be version-controlled and deployed through CI/CD.
- **Use modules/templates** for repeatable spoke deployments. Each spoke should be a parameterized template with consistent NSGs, route tables, and DNS configuration.
- **Peer and route table changes** should go through pull request review. Network misconfigurations cause outages that are hard to diagnose.

## Naming & Tagging

- **Use consistent naming conventions**: `vnet-{workload}-{env}-{region}`, `snet-{purpose}-{env}`, `nsg-{subnet}-{env}`, `rt-{subnet}-{env}`.
- **Tag all network resources** with `Environment`, `Owner`, `CostCenter`, and `Purpose`. Untagged NSGs and route tables become unmanageable at scale.

---

*Sources: [Azure networking best practices](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/plan-for-ip-addressing), [Hub-spoke reference architecture](https://learn.microsoft.com/azure/architecture/networking/architecture/hub-spoke), [Private endpoint DNS](https://learn.microsoft.com/azure/private-link/private-endpoint-dns), [Network Watcher overview](https://learn.microsoft.com/azure/network-watcher/network-watcher-overview)*
