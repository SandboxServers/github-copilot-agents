## Microsoft Defender for Cloud

### Foundational CSPM (Free)
- Unified asset inventory
- Security recommendations based on MCSB
- Secure Score — aggregated measure of security posture
- Basic Azure Policy integration

### Defender CSPM (Paid)
- **Attack path analysis** — graph-based algorithm identifying externally exploitable paths to critical assets
- **Cloud Security Explorer** — graph-based queries across the security graph for proactive risk hunting
- **Agentless scanning** — vulnerability assessment without agents, covers VMs, containers, secrets
- **Sensitive data discovery** — automatic discovery of sensitive data using Purview classification
- **Cloud Infrastructure Entitlement Management (CIEM)** — identify over-permissioned identities
- **Regulatory compliance dashboard** — track compliance against MCSB, CIS, NIST, PCI-DSS, HIPAA, SOC2

### Defender Plans by Workload
| Plan | Protects |
|---|---|
| **Defender for Servers** | VMs, Arc-enabled servers — vulnerability assessment, EDR, JIT, adaptive hardening |
| **Defender for Containers** | AKS, container registries — image scanning, runtime protection, admission control |
| **Defender for Storage** | Storage accounts — malware scanning, sensitive data threat detection, anomaly detection |
| **Defender for SQL** | Azure SQL, SQL on VMs — vulnerability assessment, threat detection, ATP |
| **Defender for App Service** | Web apps — anomaly detection, dangling DNS, attack detection |
| **Defender for Key Vault** | Key Vault — unusual access patterns, suspicious operations |
| **Defender for DNS** | DNS queries — communication with malicious domains, DNS tunneling |
| **Defender for Resource Manager** | ARM operations — suspicious management operations, lateral movement |
| **Defender for APIs** | API Management — API-specific threat detection |

### Secure Score
- Measures security posture as percentage (0-100%)
- Based on recommendations from MCSB and enabled plans
- Each recommendation has severity and potential score impact
- Track over time, target continuous improvement
- Use as KPI for security review gates
