# DNS Architecture

## Azure DNS Overview

- **Public DNS zones**: host DNS records for internet-facing domains (A, AAAA, CNAME, MX, TXT, etc.)
- **Private DNS zones**: VNet-internal name resolution — no custom DNS servers needed
- **DNS Private Resolver**: managed service replacing custom DNS forwarder VMs for hybrid resolution

## Private DNS Zones

Managed DNS for VNet-scoped name resolution. Link zones to VNets for resolution.

### Virtual Network Links
- Link private DNS zones to VNets (up to 1000 links per zone)
- **Auto-registration link**: VMs in linked VNet automatically register A records for their private IPs
- **Resolution-only link**: VNet can resolve records but VMs don't auto-register
- Only 1 private DNS zone can have auto-registration enabled per VNet
- A VNet can link to up to 1000 private DNS zones

### Resolution Order
- Default (Azure-provided DNS): private DNS zones → Azure DNS (168.63.129.16)
- Custom DNS server on VNet overrides this — must add conditional forwarder to 168.63.129.16

## Azure DNS Private Resolver

Fully managed service replacing VM-based DNS forwarders. Enables hybrid DNS resolution between Azure and on-premises.

### Inbound Endpoints
- Receive DNS queries from on-premises
- Resolve Azure private DNS zones
- Gets an IP address from a dedicated VNet subnet
- On-premises DNS servers configure conditional forwarders pointing to this IP

### Outbound Endpoints
- Forward DNS queries from Azure to on-premises or external DNS servers
- Uses DNS forwarding rulesets (up to 1000 rules)
- Rulesets can be linked directly to VNets

### Subnet Requirements
- Dedicated subnets with delegation: `Microsoft.Network/dnsResolvers`
- Separate subnets for inbound and outbound endpoints

## Hybrid DNS Architecture

### On-Premises → Azure Resolution
```
On-prem DNS server
  → conditional forwarder (privatelink.*.core.windows.net)
    → Azure DNS Private Resolver inbound endpoint
      → private DNS zones → private IP
```

### Azure → On-Premises Resolution
```
Azure VNet
  → DNS Private Resolver outbound endpoint
    → forwarding ruleset (corp.contoso.com → on-prem DNS IP)
      → on-prem DNS server → on-prem records
```

### Alternative: Custom DNS Forwarder VMs
- Deploy DNS forwarder VMs in hub VNet (Windows DNS or BIND)
- Configure conditional forwarding to 168.63.129.16 for Azure zones
- Configure forwarding to on-premises DNS for corporate zones
- Less preferred — DNS Private Resolver is managed and simpler

## Private Endpoint DNS

Each PaaS service type requires its own private DNS zone for private endpoint resolution.

### Common Zone Names
- Blob Storage: `privatelink.blob.core.windows.net`
- SQL Database: `privatelink.database.windows.net`
- Key Vault: `privatelink.vaultcore.azure.net`
- Cosmos DB: `privatelink.documents.azure.com`
- ACR: `privatelink.azurecr.io`
- App Service: `privatelink.azurewebsites.net`

### How It Works
- CNAME chain: `account.blob.core.windows.net` → `account.privatelink.blob.core.windows.net` → A record → private IP
- Zone must be linked to VNet(s) that need to resolve the private endpoint
- Centralize zones in hub VNet for hub-spoke topologies
- Use Azure Policy to auto-create DNS records when private endpoints are provisioned

### Gotchas
- Without correct DNS zone, clients resolve the public IP even though private endpoint exists
- Custom DNS servers need conditional forwarder for `privatelink.*` zones → 168.63.129.16
- Don't put records for multiple services in the same private DNS zone

## Best Practices

- Centralize all private DNS zones in hub VNet
- Use DNS Private Resolver instead of custom forwarder VMs for new deployments
- Deploy resolvers in 2+ regions for failover
- Use Azure Policy to enforce private DNS zone creation and linking
- Document all DNS forwarding rules and zone links
- Test resolution from all network paths: VNet, peered VNets, on-premises
