# Firewalling and WAF

### Azure Firewall
- Managed, cloud-native, stateful firewall-as-a-service
- L3-L7 filtering — network rules (L3/L4), application rules (L7 FQDN filtering), NAT rules (DNAT)
- SKUs: Standard, Premium (TLS inspection, IDPS, URL filtering, web categories), Basic
- Deployed in dedicated `AzureFirewallSubnet` (/26 minimum)
- Integrates with Firewall Manager for centralized policy across hubs/VWANs
- **Gotcha**: NSGs not required on AzureFirewallSubnet (managed by platform)
- **Best for**: centralized outbound/east-west filtering, FQDN-based rules, threat intelligence

### Azure Firewall vs NVAs
- Firewall: managed service, no patching, built-in HA, auto-scale, integrated with Azure
- NVAs (Palo Alto, Fortinet, etc.): bring existing licensing/policies, deeper inspection features, existing team expertise
- **Use NVA when**: organization has existing investment, needs features Azure Firewall doesn't offer, regulatory requirement for specific vendor
- **Use Azure Firewall when**: want managed service, don't need advanced NVA features, want simpler operations

### Azure Front Door WAF vs Application Gateway WAF
- **Front Door WAF**: global, edge-based, L7, HTTP/S only, DDoS at edge, caching
- **Application Gateway WAF**: regional, VNet-deployed, L7, HTTP/S, supports internal/private apps
- **Azure Firewall**: L3-L7, all protocols, centralized east-west and outbound
- Front Door WAF for public-facing web apps → Application Gateway WAF for internal apps → Azure Firewall for everything else

### DDoS Protection
- **Basic**: free, automatic, Azure infrastructure-level L3/L4 protection
- **Standard**: per-VNet, enhanced mitigation, adaptive tuning, attack analytics, cost protection
- Supported on: VMs, Application Gateway, Load Balancer, Bastion, Firewall, VPN Gateway
- **Not supported on**: Virtual WAN, PaaS multitenant (App Service), NAT Gateway
