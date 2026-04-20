# Threat Modeling

STRIDE is the primary threat modeling framework for Azure architecture reviews.

## STRIDE Model

Each letter represents a threat category:

- **Spoofing** — Can an attacker impersonate a user or system? Mitigation: strong authentication (Entra ID, MFA, managed identity). Verify every identity at every boundary.
- **Tampering** — Can an attacker modify data, code, or configuration? Mitigation: immutable storage, code signing, integrity monitoring, RBAC on write operations.
- **Repudiation** — Can a user deny performing an action with no way to prove otherwise? Mitigation: comprehensive logging (Activity Log, diagnostic logs, Sentinel). Tamper-proof log storage.
- **Information Disclosure** — Can sensitive data leak to unauthorized parties? Mitigation: encryption at rest and in transit, private endpoints, RBAC, DLP policies, data classification.
- **Denial of Service** — Can an attacker make the service unavailable? Mitigation: DDoS Protection Standard, WAF, autoscaling, rate limiting, geo-distribution.
- **Elevation of Privilege** — Can an attacker gain access beyond their authorization? Mitigation: least privilege RBAC, PIM, conditional access, network segmentation.

## Applying STRIDE to Azure Components

For every Azure component in the architecture, systematically ask:

1. What identities can be spoofed? (Service principals, managed identities, user accounts)
2. What data or configuration can be tampered with? (Storage blobs, database records, ARM templates)
3. What actions aren't logged? (Check diagnostic settings, Activity Log coverage)
4. What data can leak? (Public endpoints, overly broad RBAC, unencrypted traffic)
5. What can be overwhelmed? (Single-region deployment, no autoscaling, no WAF)
6. What can escalate privilege? (Over-permissioned identities, missing conditional access)

## Common Azure Threat Scenarios

| Scenario | STRIDE Category | Risk | Mitigation |
|---|---|---|---|
| Public storage account with anonymous access | Information Disclosure | Data breach, regulatory violation | Private endpoint, disable anonymous access, RBAC |
| Over-permissioned service principal (Owner at subscription) | Elevation of Privilege | Full environment compromise | Scope to resource group, use minimal built-in role |
| No WAF on internet-facing App Service | Tampering, DoS | Application attack, data manipulation | Application Gateway with WAF v2, OWASP rules |
| No diagnostic logs on Key Vault | Repudiation | Cannot audit secret access | Enable diagnostic settings, send to Log Analytics |
| Shared access keys instead of Entra auth | Spoofing | Unattributed access, key compromise | Disable shared key auth, use managed identity |
| No MFA on admin accounts | Spoofing | Account takeover | Conditional access requiring MFA for all admins |

## Data Flow Analysis

For every architecture, map data flows between components:

1. **Identify all data movements** — user to app, app to database, app to storage, service to service, on-prem to cloud
2. **Mark trust boundaries** — VNet boundary, internet boundary, on-prem boundary, subscription boundary, tenant boundary
3. **Verify controls at each boundary crossing** — every trust boundary crossing needs authentication, authorization, and encryption
4. **Check for sensitive data** — PII, PHI, financial data, secrets — trace their path through the system
5. **Validate encryption** — TLS 1.2+ in transit, AES-256 at rest, key management via Key Vault

## Threat Mitigation Mapping

Every identified threat must have:

- **A specific control** — not "we'll add encryption" but "TLS 1.2 enforced via minimum TLS version on Storage account"
- **Verification that control is implemented** — policy assignment, ARM template setting, Defender for Cloud recommendation status
- **Defense in depth** — multiple controls per threat (network isolation AND encryption AND access control)
- **Residual risk assessment** — what risk remains after controls? Is it accepted, transferred, or needs further mitigation?

## Threat Model Output

The deliverable is a threat model document containing:
- Architecture diagram with trust boundaries marked
- STRIDE analysis for each component
- Data flow diagram with encryption status
- Threat/mitigation matrix
- Residual risk register
- Action items with owners and timelines
