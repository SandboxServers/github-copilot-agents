# Azure Networking Anti-Patterns

## Topology & Architecture

### Flat network topology — no segmentation
Deploying all resources into a single VNet with no subnets or NSGs. Every resource can reach every other resource. A compromised VM can scan and attack the entire network. Use hub-spoke with NSGs on every subnet to enforce least-privilege.

### Single hub without redundancy
Running one hub VNet with a single Azure Firewall or NVA. If the hub fails, all spoke connectivity is lost. Deploy hub resources across Availability Zones and consider multi-region hub-spoke for disaster recovery.

### Overlapping IP address spaces across VNets or on-premises
Using the same CIDR ranges (e.g., 10.0.0.0/24) in multiple VNets or on-premises networks. VNet peering and VPN connections fail or produce unpredictable routing when address spaces overlap. Document all ranges in a central IPAM and enforce non-overlapping allocation.

### Not planning IP address space for growth
Allocating tiny subnets (/28 or /27) that run out of IPs when services scale. Azure reserves 5 IPs per subnet. A /28 gives only 11 usable addresses. Start with /16 for VNets and /24 for subnets unless you have a specific reason to constrain.

## Security & Access Control

### Public endpoints when private endpoints are available
Exposing Storage Accounts, SQL Databases, Key Vaults, and other PaaS services on public internet when Private Endpoint support exists. This expands attack surface unnecessarily. Use private endpoints and disable public network access on the resource.

### NSG rules that are too permissive
Rules like "Allow Any Inbound from Any Source on Any Port" defeat the purpose of network security groups entirely. Start with deny-all inbound, then add specific allow rules with source IP/service tag, destination port, and protocol. Review rules quarterly.

### Not tagging NSGs with purpose and owner
NSGs named `nsg1`, `nsg2` with no tags. When troubleshooting, no one knows which NSG protects which workload. Tag every NSG with `Purpose`, `Owner`, `Environment`, and associate with a specific subnet or NIC.

### NSG rules using hardcoded IPs instead of service tags
Manually maintaining IP lists for Azure services that change regularly. Use service tags (`AzureCloud`, `Storage`, `Sql`, `AzureMonitor`, etc.) which are automatically updated by Microsoft.

## Routing & Traffic Flow

### Using UDRs when Azure Firewall rules would be simpler
Creating complex UDR chains across multiple subnets when Azure Firewall application and network rules can handle the filtering centrally. UDRs control *where* traffic goes; firewall rules control *what* traffic is allowed. Use both appropriately — don't substitute one for the other.

### Ignoring effective routes when troubleshooting
Checking only the route table attached to a subnet and missing system routes, BGP-propagated routes, or routes from other sources. Always use Network Watcher → Next Hop or the "Effective Routes" blade on a NIC to see what the VM actually uses.

## PaaS & Private Connectivity

### Not using service endpoints or private endpoints for PaaS
Letting traffic to Azure Storage, SQL, or Key Vault traverse the public internet when it could stay on the Azure backbone. Service endpoints are free and keep traffic on the Microsoft network. Private endpoints give you a private IP in your VNet.

### DNS configuration as afterthought — breaks private endpoints
Deploying private endpoints without configuring Private DNS Zones. The endpoint gets a private IP, but DNS still resolves to the public IP. Connectivity fails silently — the client connects to the public endpoint (blocked) instead of the private one. Always create the corresponding Private DNS Zone and link it to the VNet.

## Hybrid Connectivity

### ExpressRoute without backup VPN
Relying on a single ExpressRoute circuit with no failover path. Circuit maintenance or provider outages cause total loss of hybrid connectivity. Configure S2S VPN as a backup with lower-weight BGP routes so failover is automatic.

### Not using BGP with VPN Gateway
Configuring static routes instead of BGP on site-to-site VPN connections. Static routes don't adapt to network changes and require manual updates. BGP enables dynamic route exchange and automatic failover.

## Operational

### No Network Watcher enabled
Deploying resources in a region without enabling Network Watcher. NSG flow logs, connection troubleshoot, packet capture — all require Network Watcher. Enable it in every region where you have resources.

### Not monitoring NSG flow logs or VNet flow logs
Deploying NSGs without enabling flow logs. You have no visibility into what traffic is allowed or denied. Enable VNet flow logs (preferred) or NSG flow logs and send to Log Analytics for analysis and alerting.

### No DDoS Protection on public-facing VNets
VNets hosting public endpoints without Azure DDoS Protection Standard. Basic DDoS protection is automatic but limited. Standard provides enhanced mitigation, telemetry, alerting, and cost protection for public IP resources.

### Deploying Application Gateway without WAF
Using Application Gateway as a pure load balancer for internet-facing web apps without enabling WAF. This misses layer-7 protection against OWASP Top 10 attacks. Use Application Gateway WAF v2 or Azure Front Door with WAF for web workloads.

### Manual network changes without Infrastructure as Code
Making network changes (NSGs, route tables, peerings) directly in the portal. Changes are untracked, unreviewable, and unreproducible. Define all networking in Bicep or Terraform, deploy through CI/CD, and treat network config as code.

---

*Sources: [Azure networking best practices](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/plan-for-ip-addressing), [Hub-spoke topology](https://learn.microsoft.com/azure/architecture/networking/architecture/hub-spoke), [Private endpoint DNS](https://learn.microsoft.com/azure/private-link/private-endpoint-dns)*
