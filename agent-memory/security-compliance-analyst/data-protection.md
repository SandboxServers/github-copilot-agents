# Data Protection Review

Data protection ensures confidentiality, integrity, and proper handling of data throughout its lifecycle.

## Data Classification

All data must be classified before controls can be applied:

| Classification | Description | Required Controls |
|---|---|---|
| **Public** | No business impact if disclosed | No special controls. Standard encryption at rest. |
| **Internal** | General business data, not for public consumption | Encryption at rest (PMK), TLS in transit, RBAC access control |
| **Confidential** | Business-sensitive, contractual obligations | CMK encryption, private endpoints, access logging, DLP policies |
| **Highly Confidential** | Regulated data (PII, PHI, PCI), trade secrets | CMK + double encryption, private endpoints, audit all access, immutable logs, data residency controls |

## Encryption at Rest

- **Platform-managed keys (PMK)** — default for all Azure services. AES-256. Acceptable for Public and Internal data.
- **Customer-managed keys (CMK)** — keys in Key Vault, customer controls rotation and access. Required for Confidential and Highly Confidential data.
- **Double encryption** — infrastructure encryption layer plus service encryption layer. Available for Storage, SQL, Cosmos DB. Required for Highly Confidential where available.
- **Key Vault HSM** — Premium tier uses FIPS 140-2 Level 2 HSMs. Managed HSM for Level 3. Required for regulated workloads.

## Encryption in Transit

- **TLS 1.2 minimum** — enforced on all services via minimum TLS version setting. TLS 1.0 and 1.1 are findings.
- **Enforce HTTPS** — disable HTTP access on Storage accounts, App Services, Function Apps. Redirect HTTP to HTTPS.
- **Certificate management** — App Service managed certificates for custom domains, Key Vault certificates for infrastructure, automate renewal.
- **Internal traffic** — mTLS for service-to-service where possible. Service mesh (Istio, Linkerd) for microservices. At minimum, TLS for all internal communication.

## Key Management

- **Azure Key Vault for all keys, secrets, and certificates** — centralized, audited, access-controlled
- **RBAC access model** — use Key Vault RBAC roles (Secrets Officer, Crypto Officer), not legacy access policies
- **Soft delete enabled** — mandatory since February 2025, 90-day retention
- **Purge protection for production** — prevent permanent deletion during soft delete period
- **Backup keys** — regularly back up Key Vault keys and secrets. Test restoration.
- **Rotation policy** — configure automatic rotation for keys. Notification before expiry.

## Data Residency

- **Understand regulatory requirements** — GDPR (EU), data sovereignty laws, industry regulations dictating where data can be stored
- **Azure region selection** — choose regions that satisfy residency requirements. Document the decision.
- **Data replication regions** — GRS replicates to paired region. Understand where paired regions are. Use ZRS or LRS if cross-region replication is not allowed.
- **Sovereign clouds** — Azure Government for US government workloads. Azure China (21Vianet). Azure for specific regulatory environments.

## Immutable Storage

- **WORM (Write Once, Read Many)** — for regulatory compliance (SEC 17a-4, FINRA, CFTC). Data cannot be modified or deleted during retention period.
- **Legal hold** — for litigation. No time limit. Data preserved until hold is removed.
- **Time-based retention** — set retention period. Data cannot be deleted until period expires. Can be locked (immutable policy).
- **Audit trail** — all immutability policy changes are logged. Compliance auditors can verify via Activity Log.
