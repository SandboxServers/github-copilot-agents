# Microsoft Defender for Cloud

Defender for Cloud is the primary security posture management and workload protection tool for Azure.

## CSPM — Cloud Security Posture Management

CSPM identifies misconfigurations and provides recommendations:

- **Security recommendations** — prioritized list of security improvements based on Microsoft Cloud Security Benchmark (MCSB)
- **Secure score** — aggregated percentage (0-100%) measuring overall security posture. Track weekly, use as KPI for security gates.
- **Attack path analysis** — graph-based algorithm identifying externally exploitable paths to critical assets (Defender CSPM tier)
- **Cloud security explorer** — query the security graph to proactively hunt for risks across your environment
- **Regulatory compliance dashboard** — track compliance against CIS, NIST 800-53, PCI DSS, HIPAA, MCSB

## CWP — Cloud Workload Protection

Defender plans protect specific workload types:

| Plan | Key Capabilities |
|---|---|
| **Defender for Servers** | Vulnerability assessment (Qualys/MDVM), JIT VM access, file integrity monitoring, EDR (MDE integration), adaptive application controls |
| **Defender for Containers** | Container image scanning in registry, runtime threat protection, Kubernetes admission control, binary drift detection |
| **Defender for Storage** | Malware scanning on upload, anomaly detection (unusual access patterns), sensitive data discovery |
| **Defender for SQL** | Vulnerability assessment, advanced threat protection (SQL injection, brute force, anomalous access) |
| **Defender for Key Vault** | Unusual access patterns, access from suspicious IPs, unusual operations (bulk retrieval, policy changes) |
| **Defender for App Service** | Anomaly detection, dangling DNS detection, communication with suspicious IPs |
| **Defender for DNS** | Communication with known malicious domains, DNS tunneling, data exfiltration over DNS |
| **Defender for Resource Manager** | Suspicious management operations, extension installation, lateral movement via ARM |

## Key Recommendations to Track

- Recommendations are prioritized by severity (High, Medium, Low) and grouped by MITRE ATT&CK tactics
- Quick fix available for many recommendations — one-click remediation
- Track remediation progress over time. Assign owners for each recommendation.
- Export recommendations to Azure Resource Graph for custom reporting and dashboards

## Workload Protection Alerts

Defender generates alerts for active threats:

- **Brute force attacks** — repeated failed authentication attempts against VMs, SQL, Key Vault
- **Crypto mining** — unusual compute patterns indicating cryptocurrency mining
- **Anomalous access** — access from unusual locations, times, or IP addresses
- **Lateral movement** — suspicious management operations attempting to spread through environment
- **Data exfiltration** — unusual data transfer volumes, DNS tunneling

Each alert includes: severity, affected resource, description, remediation steps, MITRE ATT&CK mapping.

## Best Practices

- **Enable foundational CSPM** (free tier) on all subscriptions at minimum — there's no reason not to
- **Enable Defender plans for all production workloads** — Servers, Containers, Storage, SQL, Key Vault at minimum
- **Configure email notifications** — security team must receive high-severity alerts immediately
- **Integrate with Sentinel** — stream Defender alerts to Sentinel for SIEM/SOAR correlation and automated response
- **Review secure score weekly** — track trends, investigate drops, celebrate improvements
- **Suppress false positives** — create alert suppression rules for known benign activities to reduce noise
- **Enable continuous export** — export recommendations and alerts to Log Analytics for custom queries and long-term retention
