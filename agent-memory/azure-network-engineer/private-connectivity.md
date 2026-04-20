# Private Connectivity

## Private Endpoints (Private Link)

PaaS service gets a dedicated private IP address from your VNet address space. All traffic stays on the Microsoft backbone — never touches the public internet.

### How Private Endpoints Work
- A private endpoint creates a NIC with a private IP mapped to a specific PaaS instance
- Per-instance isolation — built-in data exfiltration protection
- Works with: Storage, SQL Database, Cosmos DB, Key Vault, App Service, ACR, Event Hubs, Service Bus, and many more
- PaaS resource can disable public IP entirely after private endpoint is configured
- Accessible from on-premises via VPN or ExpressRoute

### DNS Requirements
- Requires a private DNS zone for the service (e.g., `privatelink.blob.core.windows.net`)
- CNAME chain: `storageaccount.blob.core.windows.net` → `storageaccount.privatelink.blob.core.windows.net` → private IP
- Without correct DNS zone linked to VNet, clients resolve the public IP instead
- Zone must be linked to the VNet(s) that need resolution

### Cost
- Per private endpoint per hour
- Per GB of data processed through the endpoint

## Service Endpoints

Routes traffic from VNet to PaaS service via the Microsoft backbone, but the PaaS resource retains its public IP address.

### How Service Endpoints Work
- Extends VNet identity to the PaaS service — source IP becomes VNet private IP
- Applies to ALL instances of the service type (e.g., all Azure SQL servers), not per-instance
- No data exfiltration protection — traffic could reach any instance of the service
- Enable on subnet, then configure PaaS firewall to allow that VNet/subnet
- No DNS changes needed
- Free — no additional cost
- Cannot be accessed from on-premises

### Gotcha
- Enabling a service endpoint for `Microsoft.Storage` on a subnet affects ALL traffic to Azure Storage from that subnet

## Comparison Table

| Feature | Private Endpoint | Service Endpoint |
|---|---|---|
| PaaS IP | Private IP in your VNet | Public IP (backbone routing) |
| DNS | Requires private DNS zone | No DNS change needed |
| On-prem access | Yes (via VPN/ExpressRoute) | No |
| Cost | Per-endpoint + data processing | Free |
| Security | Full private, no public exposure | Service still has public IP |
| Scope | Per-instance | Per-service type |
| Exfiltration protection | Yes | No |

## VNet Integration (App Service / Functions)

Outbound connectivity from App Service or Functions into a VNet subnet. Different from private endpoints (which provide inbound private access).

### How It Works
- Regional VNet Integration: same region, works with Standard v2+ and Premium App Service Plans
- Gateway-required VNet Integration: cross-region, legacy approach
- App can access resources in the VNet (VMs, private endpoints, etc.)
- Subnet gets delegated to `Microsoft.Web/serverFarms` — can't be used for other resources

### Gotchas
- VNet integration subnet and private endpoint subnet must be different subnets
- Delegated subnet cannot host other resources
- Requires sufficient IP addresses in the subnet for all plan instances

## Private DNS Zones for Private Endpoints

- Each PaaS service type has a specific private DNS zone name
- Examples: `privatelink.blob.core.windows.net`, `privatelink.database.windows.net`, `privatelink.vaultcore.azure.net`
- Centralize private DNS zones in hub VNet for hub-spoke topologies
- Link zones to all VNets that need to resolve private endpoint addresses
- Use Azure Policy to auto-create DNS records when private endpoints are created

## Best Practices

- Use private endpoints for all PaaS services in production environments
- Disable public network access on PaaS resources after private endpoints are configured
- Centralize private DNS zones in hub VNet — link to all spoke VNets
- Use Azure Policy to enforce private endpoint usage and auto-configure DNS
- Audit existing service endpoints and migrate to private endpoints where feasible
- For App Service: combine VNet integration (outbound) with private endpoint (inbound) for full isolation

## Decision Guide

```
Need private IP for PaaS resource?       → Private Endpoint
Need on-prem access to PaaS?             → Private Endpoint
Need data exfiltration protection?       → Private Endpoint
Just restrict PaaS access to VNet?       → Service Endpoint (simpler, free)
Need outbound from App Service to VNet?  → VNet Integration
Budget-constrained, basic restriction?   → Service Endpoint
```
