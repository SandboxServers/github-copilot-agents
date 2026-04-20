# Microsoft Sentinel

## SIEM + SOAR Platform

Cloud-native security information and event management (SIEM) with security orchestration, automation, and response (SOAR) built on Log Analytics.

### Core Components

| Component | Purpose | Key Features |
|-----------|---------|--------------|
| Data connectors | Ingest logs | 200+ connectors — Azure, M365, AWS, GCP, on-prem, CEF/Syslog |
| Analytics rules | Detect threats | Scheduled KQL queries, ML-based, Microsoft threat intelligence, fusion |
| Incidents | Investigate | Correlated alerts, entity mapping, investigation graph, timeline |
| Automation rules | Triage | Auto-assign, auto-close, run playbooks on incident creation/update |
| Playbooks | Respond | Logic Apps for automated response — isolate VM, disable user, create ticket |
| Workbooks | Visualize | Security dashboards, threat landscape, compliance views |
| Hunting queries | Proact | Proactive KQL threat hunting, bookmarks, livestream |
| Notebooks | Analyze | Jupyter notebooks for advanced investigation, ML-assisted hunting |

### Key Data Sources (Priority Order)

1. **Entra ID sign-in and audit logs** — identity-based attacks, impossible travel, brute force
2. **Azure Activity logs** — resource changes, deployments, RBAC modifications
3. **Defender for Cloud alerts** — security recommendations elevated to incidents
4. **Azure Firewall logs** — network-level threat indicators, denied traffic patterns
5. **Key Vault diagnostic logs** — secret/key access anomalies
6. **NSG flow logs** — network traffic analysis, lateral movement detection
7. **Office 365 audit logs** — email compromise, data exfiltration via SharePoint/OneDrive
8. **DNS Analytics** — DNS tunneling, C2 communication
9. **Third-party**: AWS CloudTrail, GCP Audit, Palo Alto, CrowdStrike

### Analytics Rule Types

| Type | Trigger | Use Case |
|------|---------|----------|
| Scheduled | KQL query on schedule (5min-14day) | Custom detection logic |
| Microsoft Security | From other Defender products | Forward Defender alerts as incidents |
| Fusion | ML correlation of multiple signals | Advanced multi-stage attack detection |
| ML Behavior Analytics | Anomaly detection on entity behavior | Impossible travel, unusual resource access |
| Threat Intelligence | IoC matching against TI feeds | Known malicious IPs, domains, file hashes |
| NRT (Near-Real-Time) | KQL runs every minute | Time-critical detections (1-min latency) |

### SOAR Patterns

Common automated responses:
- **Brute force detected** → Block IP in NSG/Firewall + create ticket + notify SOC
- **Impossible travel** → Require MFA re-auth via Conditional Access + flag incident
- **Malicious file upload** → Quarantine storage blob + notify security team
- **Compromised account** → Disable account + revoke sessions + create incident
- **Crypto mining** → Isolate VM from network + capture forensic snapshot

### Workspace Architecture

- **Single workspace**: Simpler, lower cost, cross-correlation. Best for most organizations.
- **Multi-workspace**: Regulatory requirement (data residency), MSSP scenarios, very large environments. Use workspace functions for cross-workspace queries.
- **Cost management**: Commitment tiers (100-5000 GB/day). Data collection rules to filter noise. Ingestion-time transformation. Basic logs for verbose data.

### Content Hub

Pre-built solutions from Microsoft and partners:
- Analytics rules, workbooks, hunting queries, playbooks, data connectors — bundled per product/threat
- Install relevant solutions for your environment (Azure, M365, AWS, etc.)
- Customize rules to reduce false positives
- Update solutions regularly for new detections

### Best Practices

- Enable Sentinel on the same Log Analytics workspace as Defender for Cloud
- Start with Microsoft Security analytics rules (low effort, immediate value)
- Tune scheduled rules — false positive rate should be < 10%
- Use automation rules for initial triage (auto-close known benign, auto-assign by type)
- Implement incident naming and severity standards
- Hunt proactively weekly using MITRE ATT&CK framework
- Review and update analytics rules monthly
- Monitor Sentinel costs — ingestion is the primary cost driver
