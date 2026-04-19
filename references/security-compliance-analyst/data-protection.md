## Data Protection Review

### Encryption Requirements
| Layer | Standard | Azure Implementation |
|---|---|---|
| **At rest (default)** | AES-256 | Platform-managed keys (PMK) — automatic for most services |
| **At rest (enhanced)** | AES-256 with CMK | Customer-managed keys in Key Vault, supports auto-rotation |
| **At rest (double)** | Double encryption | Infrastructure encryption + service encryption (e.g., Storage) |
| **In transit** | TLS 1.2+ | Enforced via minimum TLS version setting on each service |
| **In use** | Confidential computing | AMD SEV-SNP VMs, Intel SGX enclaves, confidential containers |
| **Key management** | FIPS 140-2 Level 2/3 | Key Vault Premium (HSM-backed) or Managed HSM |

### Data Classification and Handling
1. **Discover** — use Microsoft Purview for data discovery and classification across Azure, M365, on-prem
2. **Classify** — sensitivity labels (Public, Internal, Confidential, Highly Confidential)
3. **Protect** — DLP policies based on classification, encryption required for Confidential+
4. **Monitor** — audit access to sensitive data, alert on anomalous access patterns
5. **Retain** — retention policies aligned to regulatory requirements, immutable storage for compliance

### PII Handling
- Identify PII in data stores using Purview or SQL Data Discovery & Classification
- Apply Dynamic Data Masking (DDM) for non-privileged access
- Encrypt PII at rest with customer-managed keys
- Audit all access to PII-containing resources
- Define retention periods per regulatory requirement (GDPR: right to erasure, HIPAA: 6 years)
