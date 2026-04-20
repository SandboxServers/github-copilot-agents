# Security Frameworks and Benchmarks

## Microsoft Cloud Security Benchmark (MCSB)

Primary security configuration baseline for Azure. Prescriptive best practices organized into security control domains. Maps to CIS Controls, NIST SP 800-53, and PCI-DSS. Version 2 introduces AI Security domain and 420+ Azure Policy built-in definitions.

### Security Control Domains

| Domain | ID Prefix | Focus | Key Controls |
|--------|-----------|-------|--------------|
| Network Security | NS | VNet, private endpoints, NSGs, DDoS, firewall, WAF | NS-1: Network segmentation, NS-2: Secure cloud services with network controls |
| Identity Management | IM | Entra ID, MFA, conditional access, managed identity | IM-1: Use centralized identity, IM-3: Manage app identities securely |
| Privileged Access | PA | Admin workstations, JIT, emergency access, separation of duties | PA-1: Separate admin users, PA-7: Just-enough administration |
| Data Protection | DP | Encryption at rest/in transit/in use, key management, DLP | DP-1: Discover and classify sensitive data, DP-4: Enable at-rest encryption |
| Asset Management | AM | Inventory, tagging, authorized services, secure lifecycle | AM-1: Track asset inventory, AM-2: Use approved services only |
| Logging & Threat Detection | LT | Diagnostic settings, SIEM, audit logging, threat detection | LT-1: Enable threat detection, LT-4: Enable network logging |
| Incident Response | IR | Preparation, detection, containment, post-incident | IR-1: Preparation, IR-2: Detection and analysis |
| Posture & Vulnerability Mgmt | PV | Secure config, vulnerability scanning, patching | PV-1: Define secure configurations, PV-5: Perform vulnerability assessments |
| Endpoint Security | ES | EDR, anti-malware, OS hardening | ES-1: Use EDR, ES-2: Use modern anti-malware |
| Backup and Recovery | BR | Backup coverage, RBAC on backups, encryption | BR-1: Ensure regular automated backups, BR-2: Protect backup data |
| DevOps Security | DS | CI/CD security, SBOM, artifact signing, dependencies | DS-1: Conduct threat modeling, DS-6: Enforce workload security |
| Governance & Strategy | GS | Roles, segmentation strategy, security architecture | GS-1: Align roles and responsibilities, GS-2: Define security strategy |
| AI Security | AI | Model security, prompt injection, data grounding | AI-1: Map AI data flow, AI-3: Secure AI model configuration |

### Using MCSB

- **Azure Policy**: Built-in initiative "Microsoft Cloud Security Benchmark" — assign at management group level
- **Defender for Cloud**: Regulatory compliance blade maps recommendations to MCSB controls
- **Assessment**: Review each domain, identify gaps, prioritize by severity and blast radius
- **Mapping**: MCSB maps to CIS, NIST, PCI-DSS — use MCSB as single source, cross-reference for auditors

## CIS Azure Foundations Benchmark

Azure-specific implementation of CIS Controls. Two levels:
- **Level 1**: Essential security hygiene, minimal impact on functionality. Every environment.
- **Level 2**: Defense-in-depth, may impact usability. High-security environments.

Key sections: Identity and Access Management, Microsoft Defender, Storage Accounts, Database Services, Logging and Monitoring, Networking, Virtual Machines, Key Vault, App Service.

Implementation: Azure Policy built-in initiative "CIS Microsoft Azure Foundations Benchmark". Audit mode first, then enforce.

## NIST SP 800-53

Comprehensive security and privacy controls for federal information systems. Control families:

| Family | ID | Focus |
|--------|-----|-------|
| Access Control | AC | Account management, least privilege, session controls |
| Audit & Accountability | AU | Audit events, content, storage, review |
| Configuration Management | CM | Baseline config, change control, least functionality |
| Identification & Authentication | IA | MFA, authenticator management, re-authentication |
| Incident Response | IR | Policy, handling, monitoring, reporting |
| System & Communications | SC | Boundary protection, transmission confidentiality, cryptographic key mgmt |
| System & Information Integrity | SI | Flaw remediation, malware protection, monitoring |

Impact levels: Low, Moderate, High. Azure regulatory compliance dashboard tracks NIST compliance status. FedRAMP builds on NIST 800-53 with additional requirements.

## Framework Mapping Strategy

Use MCSB as primary framework. Map to others for compliance evidence:
- MCSB control → CIS benchmark recommendation → NIST 800-53 control
- Single implementation, multiple compliance reports
- Azure Policy initiatives provide built-in mapping
- Defender for Cloud regulatory compliance dashboard shows multi-framework status simultaneously

## Practical Application

1. **New workload**: Start with MCSB assessment. Identify applicable compliance frameworks. Assign Azure Policy initiatives. Review Defender for Cloud recommendations.
2. **Audit preparation**: Pull regulatory compliance report from Defender for Cloud. Cross-reference with specific framework requirements. Document compensating controls for any gaps.
3. **Continuous compliance**: Azure Policy audit → Defender for Cloud recommendations → Remediation tasks → Track improvement over time.
