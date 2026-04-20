# Compliance Frameworks

Understanding what each framework actually requires and how Azure supports it.

## SOC 2

Service Organization Control 2 — most common audit for SaaS/cloud companies.

- **Trust service criteria** — Security (mandatory), Availability, Processing Integrity, Confidentiality, Privacy (choose applicable ones)
- **Type I vs Type II** — Type I evaluates control design at a point in time. Type II evaluates operating effectiveness over a period (typically 12 months). Type II is what customers want.
- **Azure provides** — SOC 2 Type II report via Service Trust Portal. Covers Azure platform controls.
- **Customer responsibility** — your own access management, change management, monitoring, incident response controls on top of Azure
- **Common gaps** — missing quarterly access reviews, incomplete change management documentation, incident response plan not tested annually

## HIPAA

Health Insurance Portability and Accountability Act — protects PHI (Protected Health Information).

- **Requirements** — administrative safeguards, physical safeguards, technical safeguards. Business Associate Agreements (BAA) with all vendors handling PHI.
- **BAA with Microsoft required** — Microsoft signs BAA for covered Azure services. This is step one. Without BAA, cannot use Azure for PHI.
- **Technical controls** — encryption at rest (CMK for PHI), encryption in transit (TLS 1.2+), access controls (RBAC, MFA), comprehensive audit logging
- **Azure support** — HIPAA/HITRUST policy initiative in Azure Policy. Blueprint available for reference architecture.
- **Common gaps** — unencrypted PHI at rest, no audit logs on PHI-containing resources, no breach notification procedures, incomplete BAA chain with subcontractors

## PCI DSS

Payment Card Industry Data Security Standard — protects cardholder data.

- **Scope** — applies to Cardholder Data Environment (CDE). Reduce scope through network segmentation and tokenization.
- **12 requirements** — network security, data protection, vulnerability management, access control, monitoring, and security policy
- **Key technical controls** — WAF, encryption of cardholder data at rest and in transit, no default passwords, logging all access to cardholder data
- **Azure support** — Azure is PCI DSS Level 1 certified. Shared Responsibility Matrix available. But Azure compliance does NOT equal your compliance.
- **Critical distinction** — you are responsible for your CDE security. Azure secures the platform, you secure your implementation.

## FedRAMP

Federal Risk and Authorization Management Program — US government cloud security.

- **Impact levels** — Low (minimal impact), Moderate (serious impact), High (severe/catastrophic impact). Based on FIPS 199 categorization.
- **Control baseline** — maps to NIST SP 800-53 controls. High = 421 controls, Moderate = 325, Low = 125.
- **Azure support** — Azure Commercial (FedRAMP High), Azure Government (FedRAMP High + DoD IL2/4/5/6)
- **Key requirements** — US data residency, US person personnel, continuous monitoring, annual assessment

## Azure Policy for Compliance

- **Built-in compliance initiatives** — CIS Microsoft Azure Foundations Benchmark, Microsoft Cloud Security Benchmark (MCSB), NIST SP 800-53, PCI DSS, HIPAA HITRUST
- **Assignment strategy** — assign at management group level for broad coverage. Use subscription-level for workload-specific additions.
- **Approach** — audit first to understand current state. Then graduate to deny/DINE for enforcement. Never start with deny.
- **Exemptions** — document justification, set expiry date, require security team approval. No permanent exemptions without risk acceptance.

## Compliance Manager

Microsoft 365 compliance posture tool (extends beyond just Azure):

- **Compliance score** — percentage-based score measuring your posture against selected frameworks
- **Improvement actions** — specific steps to improve compliance, with implementation guidance
- **Assessment templates** — pre-built templates for SOC 2, HIPAA, PCI DSS, GDPR, and many more
- **Evidence collection** — link Azure Policy results, Defender for Cloud findings, manual evidence

## Shared Responsibility Model

This is the most critical concept in cloud compliance:

- **Microsoft secures** — physical data centers, host OS, network infrastructure, hypervisor, platform services foundation
- **Customer secures** — their data, identities, access management, application configuration, network controls, endpoint protection
- **Know the boundary** — for every compliance control, determine if Microsoft covers it, you cover it, or it's shared. The shared responsibility matrix for each framework spells this out.
