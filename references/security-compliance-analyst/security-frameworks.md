## Security Frameworks and Benchmarks

### Microsoft Cloud Security Benchmark (MCSB)
The primary security configuration baseline for Azure. Provides prescriptive best practices organized into security control domains. Maps to CIS Controls, NIST SP 800-53, and PCI-DSS. Version 2 (preview) introduces AI Security domain and 420+ Azure Policy built-in definitions.

#### Security Control Domains
| Domain | Focus |
|---|---|
| **Network Security (NS)** | VNet configuration, private endpoints, NSGs, DDoS protection, firewall, WAF |
| **Identity Management (IM)** | Entra ID, MFA, conditional access, PIM, managed identity, least privilege |
| **Privileged Access (PA)** | Admin workstations, JIT access, emergency access, separation of duties |
| **Data Protection (DP)** | Encryption at rest/in transit/in use, key management, data classification, DLP |
| **Asset Management (AM)** | Inventory, tagging, authorized services, secure lifecycle |
| **Logging and Threat Detection (LT)** | Diagnostic settings, SIEM integration, audit logging, threat detection |
| **Incident Response (IR)** | Preparation, detection, containment, post-incident activity |
| **Posture and Vulnerability Management (PV)** | Secure configurations, vulnerability scanning, patching, red team |
| **Endpoint Security (ES)** | EDR, anti-malware, OS hardening |
| **Backup and Recovery (BR)** | Backup coverage, RBAC on backups, encryption, validation |
| **DevOps Security (DS)** | CI/CD pipeline security, SBOM, artifact signing, dependency scanning |
| **Governance and Strategy (GS)** | Roles and responsibilities, segmentation strategy, security architecture |
| **AI Security** | AI model security, prompt injection protection, data grounding security |

### CIS Controls v8
20 critical security controls organized by implementation group (IG1/IG2/IG3). Maps to MCSB controls. Azure CIS Benchmark provides specific Azure configuration guidance for each control. Focus areas: inventory, data protection, secure configuration, access control, audit logging, incident response.

### NIST SP 800-53
Comprehensive security and privacy controls for federal information systems. Organized into families: AC (Access Control), AU (Audit), CA (Assessment), CM (Configuration), IA (Identification), IR (Incident Response), SC (System Communications), SI (System Integrity). Azure regulatory compliance dashboard tracks NIST compliance status.
