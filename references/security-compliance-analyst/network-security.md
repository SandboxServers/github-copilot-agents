## Network Security Review

### Network Exposure Assessment
| Check | Secure | Insecure |
|---|---|---|
| **Public endpoints** | Disabled or restricted to specific IPs/VNets | Open to internet (0.0.0.0/0) |
| **Private endpoints** | Used for PaaS services (Storage, SQL, Key Vault, Cosmos DB) | Service endpoints only or public access |
| **NSG rules** | Deny by default, allow specific ports/sources | Allow all inbound from any source |
| **TLS version** | TLS 1.2+ enforced | TLS 1.0/1.1 permitted |
| **SSH/RDP** | Via Azure Bastion or JIT VM access | Direct public exposure on port 22/3389 |
| **DNS** | Azure Private DNS zones for private endpoints | Public DNS resolution for internal services |
| **WAF** | Enabled on Application Gateway / Front Door | No WAF on internet-facing apps |
| **DDoS Protection** | Standard plan on production VNets | No DDoS protection |

### Traffic Encryption Requirements
- **In transit**: TLS 1.2+ for all external connections. Consider mTLS for service-to-service.
- **Internal traffic**: Encrypt east-west traffic in zero-trust networks. Use service mesh for microservices.
- **VPN/ExpressRoute**: IPSec encryption for VPN. Consider double encryption for sensitive workloads.
- **Database connections**: Always encrypted connections. Disable unencrypted access.
