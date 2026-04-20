# Azure Networking Gotchas

## Private Endpoints

### Private endpoints require Private DNS Zones — wrong DNS = silent failure
A private endpoint creates a NIC with a private IP, but DNS must resolve the service FQDN to that private IP. Without the correct Private DNS Zone (e.g., `privatelink.blob.core.windows.net` for Storage), DNS resolves to the public IP. The client connects to the public endpoint, which may be firewalled off — connectivity fails with no clear error message. Always create and link the matching Private DNS Zone.

### NSG support on private endpoints must be explicitly enabled
By default, NSG rules do NOT apply to traffic destined for private endpoints. You must set `PrivateEndpointNetworkPolicies = Enabled` on the subnet to enforce NSG rules on private endpoint traffic. Without this, your NSG allow/deny rules are bypassed for private endpoint flows.

### Private endpoints don't support NSG flow logs directly
Flow logs on the subnet won't capture private endpoint traffic unless you enable the network policy flag above. Even then, logging behavior differs from standard NIC traffic. Test your logging before relying on it.

## VNet Peering

### Peering is NOT transitive
If VNet A peers with VNet B, and VNet B peers with VNet C, VNet A **cannot** reach VNet C through VNet B. You must either: peer A↔C directly, use a hub NVA/Firewall with UDRs to route traffic, or use Virtual WAN which provides transitive routing by default.

### Peering must be created in both directions
Creating a peering from VNet A → B does not automatically create B → A. Both sides must have a peering configured. Status shows "Initiated" until the reciprocal peering exists, then both flip to "Connected."

### Removing and re-creating peering causes downtime
Deleting a peering immediately drops all traffic between the VNets. There is no graceful drain. Plan peering changes during maintenance windows.

## NSG & Security

### NSG flow logs require Network Watcher in the same region
If you deploy an NSG in East US but Network Watcher isn't enabled in East US, you cannot create flow logs for that NSG. Network Watcher is auto-enabled per subscription per region — but verify it exists before configuring flow logs.

### Default outbound access is deprecated
As of November 2024, new VMs and scale sets no longer get default outbound internet access. You must explicitly provide outbound connectivity via NAT Gateway, Load Balancer outbound rules, or a public IP on the NIC. Existing resources are grandfathered but should be migrated.

## Load Balancer

### Standard Load Balancer has no outbound by default
Unlike Basic LB, Standard LB does not provide default outbound internet connectivity. Backend pool members cannot reach the internet without: a NAT Gateway on the subnet, explicit outbound rules on the LB, or a public IP on each VM NIC.

### Health probes from 168.63.129.16 must be allowed
Azure Load Balancer health probes originate from the virtual IP `168.63.129.16`. If your NSG blocks this source, all backend instances show as unhealthy and traffic stops flowing. Always allow `AzureLoadBalancer` service tag inbound.

## Azure Firewall

### DNAT rules don't work with forced tunneling
If you configure forced tunneling (sending all internet-bound traffic to on-premises), Azure Firewall DNAT rules for inbound traffic from the internet will not function. DNAT requires the Firewall to have a direct path back to the internet source.

### Azure Firewall subnet must be named AzureFirewallSubnet
The subnet for Azure Firewall must be exactly `/AzureFirewallSubnet` with a minimum /26 prefix. Any other name causes deployment failure. No other resources can be placed in this subnet.

## Application Gateway

### Application Gateway requires a dedicated subnet
One Application Gateway per subnet. The subnet cannot contain any other resource types. Subnet sizing: /24 recommended for production to allow v2 autoscaling up to 125 instances.

### Stopping Application Gateway v2 deallocates the public IP
If you stop and restart App Gateway v2, it may get a new public IP. Use a static public IP SKU and DNS CNAME to avoid DNS update issues.

## VPN Gateway

### Resizing VPN Gateway can cause brief connectivity loss
Changing the SKU (e.g., VpnGw1 → VpnGw2) within the same generation preserves tunnels with ~30 seconds of disruption. Changing across generations or to/from Basic requires a full gateway redeploy with extended downtime.

## Hub-Spoke

### Spoke-to-spoke traffic doesn't traverse the hub automatically
VNet peering is non-transitive. Spoke-to-spoke traffic requires: (1) UDR on each spoke subnet pointing 0.0.0.0/0 or specific prefixes to the hub firewall/NVA private IP, (2) IP forwarding enabled on the NVA NIC, (3) firewall rules allowing the traffic. Without all three, spoke-to-spoke is blocked.

## Subnet Delegation

### Some delegations prevent other resources in the same subnet
Delegating a subnet to a service (e.g., `Microsoft.Web/serverFarms` for App Service VNet Integration) may prevent deploying other resource types into that subnet. Check delegation requirements before deploying — moving resources out of a delegated subnet requires delete + recreate.

## DNS

### Private DNS Zone must be linked to VNets that need resolution
Creating a Private DNS Zone doesn't automatically make it visible to all VNets. You must create a VNet link for each VNet that needs to resolve records in the zone. Missing links = DNS resolution fails for resources in those VNets.

### Custom DNS servers on VNet override Azure-provided DNS
If you set custom DNS servers on a VNet, Azure DNS (168.63.129.16) is no longer used by default. If your custom DNS doesn't forward to Azure DNS for Private DNS Zone resolution, private endpoints break. Always configure conditional forwarding to 168.63.129.16 for `privatelink.*` zones.

## AKS Networking

### AKS with Azure CNI consumes IPs rapidly
Azure CNI assigns a real VNet IP to every pod. A 10-node cluster with 30 pods per node needs 300+ IPs just for pods, plus IPs for nodes and internal LB. Size subnets accordingly — /22 or larger for production AKS clusters with Azure CNI.

---

*Sources: [Private endpoint DNS configuration](https://learn.microsoft.com/azure/private-link/private-endpoint-dns), [VNet peering](https://learn.microsoft.com/azure/virtual-network/virtual-network-peering-overview), [NSG on private endpoints](https://learn.microsoft.com/azure/private-link/disable-private-endpoint-network-policy)*
