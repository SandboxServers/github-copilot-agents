# Private Connectivity to PaaS Services

### Private Endpoints (Private Link)
- Dedicated private IP from your VNet address space mapped to specific PaaS instance
- Traffic stays on Azure backbone — never touches public internet
- Built-in data exfiltration protection — endpoint is per-instance, not per-service
- Works from on-premises via VPN/ExpressRoute
- PaaS resource can disable public IP entirely
- **Requires DNS configuration**: private DNS zone (e.g., `privatelink.blob.core.windows.net`) linked to VNet
- Cost: per endpoint hour + per GB data processed
- **Limits**: see Azure Private Link limits in subscription service limits
- **DNS gotcha**: CNAME chain — `storageaccount.blob.core.windows.net` → `storageaccount.privatelink.blob.core.windows.net` → private IP. Without correct DNS zone, resolves to public IP

### Service Endpoints
- Extends VNet identity to PaaS service — source IP becomes VNet private IP
- Traffic stays on Azure backbone but PaaS resource retains its public IP
- Applies to **all** instances of the service type (e.g., ALL Azure SQL servers), not per-instance
- No data exfiltration protection — traffic could be redirected to any instance of that service
- Free — no additional cost
- Simple setup — enable on subnet, configure service firewall rules
- No DNS changes needed
- **Cannot** be accessed from on-premises by default
- **Gotcha**: enabling service endpoint for a service (e.g., `Microsoft.Storage`) on a subnet affects ALL traffic to that service from that subnet

### VNet Integration (App Service / Functions)
- Outbound connectivity from App Service into a VNet subnet
- Different from private endpoints (which are inbound)
- Regional VNet Integration for same-region, Gateway-required VNet Integration for cross-region
- **Gotcha**: VNet integration subnet and private endpoint subnet must be different subnets
- **Gotcha**: VNet integration subnet gets delegated — can't be used for other resources

### Decision Matrix
```
Need private IP for PaaS? → Private Endpoint
Need on-prem access to PaaS? → Private Endpoint
Need data exfiltration protection? → Private Endpoint
Just want to restrict PaaS access to VNet? → Service Endpoint (simpler, free)
Need outbound from App Service to VNet? → VNet Integration
Budget-constrained, basic restriction? → Service Endpoint
```
