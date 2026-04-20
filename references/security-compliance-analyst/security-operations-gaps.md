# Security Operations Gaps

Areas where existing reference material has incomplete coverage for day-to-day security operations.

## Threat Modeling Methodology Deep Dive

The threat-modeling.md file covers STRIDE categories and common scenarios but lacks operational methodology for running threat modeling sessions at scale.

### Session Facilitation

- **Who should be in the room** — architect, lead developer, security analyst, operations representative. Product owner optional but useful for risk acceptance decisions.
- **Time-box sessions** — 90 minutes maximum per component. Longer sessions produce diminishing returns. Break complex architectures into multiple sessions.
- **Data Flow Diagrams (DFD) first** — before any STRIDE analysis, map every data flow: source, destination, protocol, data classification, trust boundary crossing. Without DFDs, threat modeling is speculation.
- **Microsoft Threat Modeling Tool** — free tool that generates threats from DFD elements automatically. Exports to Azure DevOps work items for tracking. Use the Azure template for cloud-native threat models.

### STRIDE Prioritization

Not all STRIDE categories carry equal weight for every component:

- **Internet-facing services** — Spoofing and Denial of Service are highest priority. Focus on authentication strength and availability controls.
- **Data stores** — Information Disclosure and Tampering dominate. Focus on encryption, access control, and integrity verification.
- **Internal APIs** — Elevation of Privilege and Tampering are primary concerns. Focus on authorization granularity and input validation.
- **Audit systems** — Repudiation is the critical category. Focus on tamper-proof logging and non-repudiation controls.

### Tracking and Integration

- **Threats as work items** — every identified threat becomes an Azure DevOps work item with severity, owner, and due date. Tag with `security:threat-model` for tracking.
- **Architecture change triggers** — any PR that modifies architecture diagrams, adds new external dependencies, or changes authentication flows should trigger threat model review.
- **Annual refresh** — even without architecture changes, refresh threat models annually. New attack techniques and Azure service changes may introduce threats not present in the original model.

## Compliance Framework Mapping

The compliance-frameworks.md covers individual frameworks but lacks guidance on mapping controls across frameworks and managing multi-framework compliance.

### Cross-Framework Control Mapping

Many controls satisfy requirements across multiple frameworks simultaneously:

| Control | SOC 2 | PCI-DSS | HIPAA | NIST 800-53 |
|---|---|---|---|---|
| MFA for all users | CC6.1 | Req 8.3 | §164.312(d) | IA-2 |
| Encryption at rest | CC6.1, CC6.7 | Req 3.4 | §164.312(a)(2)(iv) | SC-28 |
| Quarterly access reviews | CC6.2, CC6.3 | Req 7.2 | §164.312(a)(1) | AC-2 |
| Incident response plan | CC7.3, CC7.4 | Req 12.10 | §164.308(a)(6) | IR-1 |
| Audit logging | CC7.2 | Req 10.1-10.3 | §164.312(b) | AU-2, AU-3 |
| Vulnerability scanning | CC7.1 | Req 11.3 | §164.308(a)(1) | RA-5 |
| Network segmentation | CC6.6 | Req 1.3 | §164.312(e)(1) | SC-7 |
| Change management | CC8.1 | Req 6.5 | §164.312(e)(2)(ii) | CM-3 |

### Azure Policy Initiatives for Compliance

- **Built-in regulatory compliance initiatives** — Azure provides initiatives for PCI-DSS v4, HIPAA, NIST 800-53, SOC 2, and CIS Benchmarks. Assign these at management group scope for comprehensive coverage.
- **Custom initiative for multi-framework** — create a custom initiative that combines controls common across your required frameworks. Eliminates duplicate assessments and simplifies reporting.
- **Compliance Manager in Purview** — tracks compliance posture across Microsoft 365 and Azure. Provides improvement actions, assessment scores, and evidence collection workflows.

### Evidence Collection Automation

- **Azure Policy compliance reports** — export compliance state as evidence for auditors. Schedule weekly exports to a compliance evidence Storage account.
- **Defender for Cloud regulatory compliance dashboard** — maps security recommendations to framework controls. Screenshot or export for audit evidence.
- **Entra ID access reviews** — completed access reviews serve as evidence for SOC 2 CC6.2, PCI-DSS Req 7.2, and HIPAA access control requirements.
- **Activity Log and diagnostic logs** — retained logs serve as evidence for audit trail requirements across all frameworks. Ensure retention meets the longest framework requirement (typically PCI-DSS at 1 year, SOC 2 at 1 year, HIPAA at 6 years).

### Framework-Specific Azure Gotchas

- **PCI-DSS scope reduction** — use network segmentation and tokenization aggressively. Every resource in the CDE scope requires full PCI assessment. Private endpoints and dedicated subnets reduce scope.
- **HIPAA BAA chain** — every Azure service processing PHI must be covered by a Microsoft BAA. Check the BAA-covered services list; not all Azure services are included. Subprocessors must also have BAAs.
- **SOC 2 continuous monitoring** — SOC 2 Type II requires operating effectiveness over a period. Point-in-time fixes don't count. Implement Azure Policy with DINE effects and continuous Defender for Cloud monitoring to demonstrate ongoing compliance.
- **FedRAMP inheritance** — Azure services provide FedRAMP control inheritance. Document which controls are fully inherited, partially inherited, and customer-responsibility. Use the Azure FedRAMP Blueprint as a starting point.
