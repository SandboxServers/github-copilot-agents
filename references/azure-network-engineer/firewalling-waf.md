# Firewalling and WAF

## Azure Firewall

Managed, cloud-native, stateful firewall-as-a-service with L3–L7 filtering capabilities.

### SKUs
- **Basic**: small environments, limited features, lower cost
- **Standard**: FQDN filtering, threat intelligence, NAT rules, network rules
- **Premium**: adds TLS inspection, IDPS (intrusion detection/prevention), URL filtering, web categories

### Rule Types and Processing Order
1. **DNAT rules**: inbound NAT — translate public IP:port to private IP:port
2. **Network rules**: L3/L4 filtering by IP, port, protocol
3. **Application rules**: L7 FQDN/URL filtering — supports HTTP/S and MSSQL

Processing order: DNAT → Network → Application. First match wins within each collection.

### Deployment
- Dedicated `AzureFirewallSubnet` (/26 minimum)
- NSGs not required on AzureFirewallSubnet (managed by platform)
- Firewall Policy for centralized rule management across multiple firewalls
- Integrates with Firewall Manager for policy management across hubs and VWANs
- Structured policies: Rule Collection Groups → Rule Collections → Rules

### Key Features
- Threat intelligence: block traffic from/to known malicious IPs and domains
- DNS proxy: resolve FQDNs in network rules
- Forced tunneling support: route firewall management traffic separately
- Auto-scale: scales out based on traffic demand
- Built-in HA: no additional configuration needed

## NVA (Network Virtual Appliances)

Third-party firewalls from Azure Marketplace: Palo Alto, Fortinet, Check Point, Barracuda, etc.

### When to Use NVAs
- Existing investment in a specific vendor's platform and licensing
- Need features Azure Firewall doesn't offer (advanced threat prevention, specific compliance)
- Regulatory requirement for a specific vendor
- Team expertise with a particular product

### When to Use Azure Firewall
- Want a managed service with no patching or HA configuration
- Don't need advanced NVA-specific features
- Want simpler operations and native Azure integration
- Greenfield deployment with no existing vendor commitment

## Azure WAF (Web Application Firewall)

Layer 7 web application firewall protecting against OWASP Top 10 and other web vulnerabilities.

### WAF on Application Gateway
- Regional, VNet-deployed
- Supports internal/private applications
- Backend pool: VMs, VMSS, App Service, IPs
- OWASP Core Rule Set (CRS) 3.x
- Custom rules: IP-based, geo-based, rate limiting
- Per-site and per-listener WAF policies

### WAF on Front Door
- Global, edge-based deployment
- DDoS protection at the edge
- HTTP/S only
- Bot protection
- Rate limiting
- Geo-filtering
- Custom rules with match conditions

### Front Door WAF vs App Gateway WAF

| Feature | Front Door WAF | App Gateway WAF |
|---|---|---|
| Scope | Global, edge | Regional, VNet |
| Protocol | HTTP/S only | HTTP/S |
| DDoS | Edge-level protection | VNet-level |
| Internal apps | No (internet-facing only) | Yes |
| Caching | Yes | No |
| Backend control | Limited | Full VNet integration |

### Deployment Pattern
- Front Door WAF for public-facing web applications
- App Gateway WAF for internal/regional web applications
- Azure Firewall for all other traffic (east-west, non-HTTP)

## Best Practices

- Deploy Azure Firewall in hub VNet as central inspection point
- Route all spoke traffic through hub firewall (UDR 0.0.0.0/0 → Firewall private IP)
- WAF on all internet-facing web applications
- Start WAF in detection mode, analyze logs, then move to prevention mode
- Structure Firewall Policy with Rule Collection Groups organized by workload
- Use Firewall Manager for multi-hub and VWAN policy management
- Enable diagnostic logging on all firewall and WAF instances
- Review and update WAF rules regularly — new OWASP CRS versions
