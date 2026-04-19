# Database Security

## Defense in Depth — Azure SQL Database (Verified via MS Learn)

| Layer | Feature | What It Does |
|---|---|---|
| **Network** | Private endpoints, firewall rules, VNet service endpoints | Control who can reach the database endpoint |
| **Identity** | Microsoft Entra authentication, SQL auth | Authenticate: who are you? |
| **Access control** | RBAC, database roles, permissions | Authorize: what can you do? |
| **Encryption at rest** | TDE (Transparent Data Encryption) | Encrypt data files, logs, backups automatically |
| **Encryption in use** | Always Encrypted (+ secure enclaves) | Encrypt specific columns — even DBAs can't read plaintext |
| **Data masking** | Dynamic Data Masking | Hide sensitive data from non-privileged users on query results |
| **Row filtering** | Row-Level Security (RLS) | Restrict which rows a user can see based on execution context |
| **Column-level** | Column permissions (GRANT SELECT/UPDATE on columns) | Restrict access to sensitive columns |
| **Audit** | SQL Auditing | Track database operations to Storage, Log Analytics, or Event Hubs |
| **Threat detection** | Microsoft Defender for SQL | ML-based anomaly detection: SQL injection, brute force, unusual access |
| **Classification** | Data Discovery & Classification | Identify, classify, and label sensitive columns; integrates with Microsoft Purview |
| **Vulnerability** | SQL Vulnerability Assessment | Discover, track, remediate potential database vulnerabilities with actionable recommendations |
| **Integrity** | Ledger tables | Tamper-evident, cryptographic proof of data integrity for audit-critical data |

## TDE vs Always Encrypted (Verified via MS Learn)

| Aspect | TDE | Always Encrypted | Always Encrypted with Secure Enclaves |
|---|---|---|---|
| Scope | Entire database (files, logs, backups) | Specific columns | Specific columns |
| Encryption point | Server side (at rest only) | Client side (end-to-end) | Client side + server-side enclave processing |
| DBA can see data? | Yes (data decrypted in memory) | No (key only on client) | No (processed in secure enclave only) |
| App changes needed | None | Minimal (driver changes) | Minimal (driver changes) |
| T-SQL operations on encrypted data | Full (data decrypted in memory) | Equality comparison only (deterministic encryption) | Pattern matching, comparisons, sorting, in-place encryption |
| Key storage | Azure-managed or customer-managed (AKV) | Windows Certificate Store or Azure Key Vault | Windows Certificate Store or Azure Key Vault |
| **Use case** | Compliance — encrypt at rest | Protect data even from DBAs — strongest separation | Same as AE + richer query support on encrypted data |
| Enabled by default? | Yes (all new databases) | No (must configure) | No (must configure) |

### TDE Key Management
- Two-key hierarchy: symmetric AES-256 Database Encryption Key (DEK) encrypted by asymmetric RSA 2048 master key
- Master key options: Azure-managed (service-managed TDE) or customer-managed in Azure Key Vault (BYOK)
- Customer-managed keys provide additional control: key rotation, access policies, audit logging

### Ledger Tables (Verified via MS Learn)
- Creates immutable record of database changes — tamper-evident logging
- Cryptographic proof of data integrity via blockchain-style hash chains
- Helps meet regulatory requirements for data integrity verification
- Two types: updatable ledger tables (track changes) and append-only ledger tables (insert-only)

### Dynamic Data Masking (Verified via MS Learn)
- Masks sensitive data in query results for non-privileged users
- Data in database is NOT changed — masking happens on-the-fly
- Masking functions: default, email, random, custom string (e.g., `XXX-XX-0000` for SSN)
- Auto-discovery: automatically identifies potentially sensitive data and recommends masking
- Excluded users can see unmasked data

### Row-Level Security (Verified via MS Learn)
- Controls access at the row level based on user identity or execution context
- Filter predicates: hide rows the user shouldn't see
- Block predicates: prevent writes to rows the user shouldn't modify
- Implemented as security policy with inline table-valued functions
- Access restriction at database tier — simplifies application logic

## Authentication Best Practices

1. **Use Microsoft Entra ID** — single identity plane, conditional access, MFA, no password rotation, centralized audit
2. **Avoid SQL authentication** in production — passwords are weak, no MFA, no conditional access, no centralized management
3. **Use managed identities** for app-to-database connections — zero credential management, no secrets to rotate
4. **Entra groups** for RBAC — manage access at group level, not individual user level
5. **Regularly review access permissions** — conduct access reviews, remove unnecessary privileges, deactivate departed users
6. **Never concatenate user input into SQL** — use parameterized queries and stored procedures to prevent SQL injection

## Cosmos DB Security
- Microsoft Entra ID authentication supported
- Resource tokens for fine-grained access (per-container, per-partition key)
- IP firewall rules, VNet service endpoints, private endpoints
- Always encrypted at rest (service-managed or customer-managed keys)
- Encryption in transit (TLS)
- RBAC via Entra ID or primary/secondary keys (keys provide full access — treat as secrets)

## PostgreSQL Security
- Microsoft Entra ID authentication supported on Flexible Server
- SSL/TLS enforced by default
- Private endpoints and VNet integration
- PostgreSQL native RBAC (roles, schemas, row-level security)
- Server-level firewall rules
