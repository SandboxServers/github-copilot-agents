# Network Security Review

Network security enforces defense in depth and limits blast radius.

## Production Requirements

These are non-negotiable for production workloads:

- **Private endpoints for all PaaS services** — Storage, SQL, Cosmos DB, Key Vault, ACR, App Service. No exceptions. Public network access disabled.
- **NSGs on every subnet** — deny by default, allow specific ports and source ranges only. No NSG = no deployment.
- **No public IPs on VMs** — use Azure Bastion for management access. JIT VM access as alternative.
- **WAF on internet-facing web apps** — Application Gateway v2 with WAF or Azure Front Door with WAF. OWASP 3.2 rule set minimum.
- **DDoS Protection Standard** — on all VNets containing resources with public IPs. Basic tier is insufficient for production.

## Zero Trust Network

No implicit trust between any network segments:

- **Microsegmentation with NSGs and ASGs** — each application tier in its own subnet with its own NSG. Use Application Security Groups to simplify rules.
- **East-west traffic inspection** — Azure Firewall or NVA in hub VNet inspecting traffic between spokes. Not just north-south.
- **No implicit trust between subnets** — web tier cannot talk to database tier directly. All traffic goes through defined paths.
- **Every service-to-service call authenticated** — managed identity + RBAC, not just network proximity. Network is not an identity.

## Private Endpoint Checklist

For every PaaS service, verify:

| Service | Private Endpoint Deployed? | Public Access Disabled? | Private DNS Zone Configured? | DNS Resolution Working? |
|---|---|---|---|---|
| Storage (blob, file, table, queue) | | | | |
| Azure SQL / Cosmos DB | | | | |
| Key Vault | | | | |
| Azure Container Registry | | | | |
| App Service / Functions | | | | |
| Event Hub / Service Bus | | | | |

## Firewall Review

- **Azure Firewall or NVA deployed in hub?** — centralized egress control is required
- **All egress through firewall?** — UDR on all subnets pointing to firewall. No direct internet breakout.
- **Threat intelligence enabled?** — block known malicious IPs and domains automatically
- **TLS inspection where appropriate?** — decrypt and inspect HTTPS traffic for sensitive environments
- **IDPS in alert or deny mode?** — intrusion detection and prevention active on Azure Firewall Premium
- **Application rules over network rules** — use FQDN-based rules where possible for better control

## Common Findings (Always Flag These)

- App Service with public endpoint and no IP restriction or access restriction rules
- Storage account without private endpoint in production
- VM with a public IP address assigned directly
- NSG allowing 0.0.0.0/0 inbound on management ports (22, 3389)
- Service endpoints used where private endpoints are available (service endpoints don't disable public access)
- No WAF on internet-facing Application Gateway
- Subnets without NSG associations
- No forced tunneling — egress going directly to internet bypassing firewall
