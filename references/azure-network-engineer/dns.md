# DNS

### Azure Private DNS Zones
- Managed DNS for VNet-scoped name resolution — no custom DNS servers needed
- Link to VNets via virtual network links (up to 1000 links per zone)
- Auto-registration: VMs in linked VNet automatically get DNS records (A records for private IPs)
- Only 1 private DNS zone can have auto-registration enabled per VNet, but VNet can link to up to 1000 zones
- Resolution order (default DNS): private DNS zones → Azure-provided DNS (168.63.129.16)
- **Gotcha**: custom DNS server on VNet overrides this order — must configure conditional forwarder to 168.63.129.16 for private zone resolution

### Private Endpoint DNS
- Each Azure service has a specific private DNS zone name (e.g., `privatelink.blob.core.windows.net`, `privatelink.database.windows.net`)
- CNAME chain: public FQDN → `privatelink.*` zone → A record with private IP
- Must create private DNS zone + link to VNet for resolution to work
- **Gotcha**: without correct DNS config, clients resolve public IP even though private endpoint exists
- **Gotcha**: don't put records for multiple services in the same private DNS zone — causes deletion of initial A-record
- **Gotcha**: if custom DNS is used, need conditional forwarder for `privatelink.*` zones → Azure DNS (168.63.129.16) or DNS Private Resolver inbound endpoint

### Azure DNS Private Resolver
- Replaces VM-based DNS forwarders — fully managed service
- **Inbound endpoint**: receives queries from on-premises → resolves Azure private DNS zones (IP from VNet subnet)
- **Outbound endpoint**: forwards queries from Azure → on-premises or external DNS servers
- DNS forwarding rulesets: up to 1000 rules, applied to outbound endpoints or linked to VNets
- **Failover**: deploy resolvers in 2+ regions, configure on-premises conditional forwarders with both IPs
- Subnet delegation: `Microsoft.Network/dnsResolvers` — dedicated subnets for in/outbound endpoints

### Split-Brain DNS
- Same FQDN resolves differently for internal vs external users
- Internet users → public Azure DNS zones (public IP)
- Internal users → private DNS zones via DNS Private Resolver (private IP)
- Common for hybrid scenarios where apps are accessible both internally and externally

### DNS Troubleshooting Checklist
1. Is the private DNS zone linked to the VNet? (virtual network link)
2. Is the correct `privatelink.*` zone created for the service?
3. Does the A record exist in the private DNS zone with correct private IP?
4. Is custom DNS configured? → Need conditional forwarder to 168.63.129.16
5. Is DNS Private Resolver deployed? → Check inbound/outbound endpoint IPs
6. From on-premises: is conditional forwarder pointing to resolver inbound endpoint?
7. Test: `nslookup <fqdn>` — should return private IP, not public
