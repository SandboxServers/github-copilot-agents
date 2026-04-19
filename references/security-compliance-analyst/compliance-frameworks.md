## Compliance Frameworks — What Each Actually Requires

### SOC 2 (Type II)
- **What it is**: Trust Services Criteria — Security, Availability, Processing Integrity, Confidentiality, Privacy
- **What auditors check**: Control design AND operating effectiveness over a period (typically 12 months)
- **Key controls**: Access management, change management, risk assessment, monitoring, incident response
- **Azure support**: SOC 2 audit reports available via Service Trust Portal. Defender for Cloud regulatory compliance dashboard.
- **Common gaps**: Missing access reviews, incomplete change management documentation, no incident response testing

### HIPAA
- **What it is**: Health Insurance Portability and Accountability Act — protects PHI (Protected Health Information)
- **What's required**: Administrative, physical, and technical safeguards. Business Associate Agreements (BAA).
- **Key controls**: Access controls, audit controls, integrity controls, transmission security, encryption
- **Azure support**: Microsoft signs BAA for covered services. HIPAA/HITRUST policy initiative in Azure Policy.
- **Common gaps**: Unencrypted PHI at rest, missing audit logs, no breach notification procedures, incomplete BAA chain

### PCI-DSS (v4.0)
- **What it is**: Payment Card Industry Data Security Standard — protects cardholder data
- **What's required**: 12 requirements across 6 goals (network security, data protection, vulnerability management, access control, monitoring, policy)
- **Key controls**: Firewall/WAF, encryption of cardholder data, strong access control, regular testing, security policy
- **Azure support**: Azure certified PCI-DSS Level 1 (>6M transactions/year). Shared Responsibility Matrix available.
- **Critical**: Azure compliance doesn't equal YOUR compliance. You're responsible for your CDE (Cardholder Data Environment).

### FedRAMP
- **What it is**: Federal Risk and Authorization Management Program — standardized approach for cloud security assessment
- **Levels**: Low, Moderate, High — based on data sensitivity (FIPS 199 categorization)
- **What's required**: NIST SP 800-53 controls mapped to impact level. Continuous monitoring.
- **Azure support**: Azure (FedRAMP High), Azure Government (FedRAMP High + DoD IL2/4/5/6). System Security Plan available.
- **Key considerations**: US data residency requirements, personnel security, continuous monitoring reporting
