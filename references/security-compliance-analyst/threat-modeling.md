## Threat Modeling

### STRIDE Framework
| Threat | Description | Azure Mitigation |
|---|---|---|
| **Spoofing** | Impersonating a user or system | Entra ID authentication, MFA, managed identity |
| **Tampering** | Modifying data or code | Immutable storage, code signing, integrity monitoring |
| **Repudiation** | Denying actions without proof | Azure Monitor audit logs, Sentinel, Activity Log |
| **Information Disclosure** | Exposing data to unauthorized parties | Encryption, private endpoints, RBAC, DLP |
| **Denial of Service** | Making service unavailable | DDoS Protection, WAF, autoscaling, rate limiting |
| **Elevation of Privilege** | Gaining unauthorized access | Least privilege, PIM, conditional access, RBAC |

### Architecture Attack Surface Checklist
1. **Internet-facing components** — Application Gateway/Front Door, public IPs, DNS entries
2. **Authentication boundaries** — how do users/services authenticate? Any anonymous access?
3. **Data flows** — where does sensitive data move? Is it encrypted in every hop?
4. **Administrative access** — how do operators access production? Bastion? JIT? VPN?
5. **Supply chain** — container images, NuGet/npm packages, Terraform modules — trusted sources?
6. **Secrets management** — where are connection strings, API keys, certificates? Key Vault?
7. **Cross-tenant/cross-subscription** — any shared resources? How is isolation maintained?
8. **Backup and recovery** — are backups encrypted? Can an attacker delete backups?
