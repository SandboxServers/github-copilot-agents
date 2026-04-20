# Database Security

## Defense in Depth Layers

| Layer | Feature | What It Does |
|---|---|---|
| Network | Private endpoints, firewall rules | Control who can reach the database endpoint |
| Identity | Entra ID authentication, SQL auth | Authenticate: who are you? |
| Access control | RBAC, database roles, permissions | Authorize: what can you do? |
| Encryption at rest | TDE (Transparent Data Encryption) | Encrypt data files, logs, backups automatically |
| Encryption in use | Always Encrypted (+ secure enclaves) | Encrypt specific columns — even DBAs cannot read plaintext |
| Data masking | Dynamic Data Masking | Hide sensitive data from non-privileged users |
| Row filtering | Row-Level Security (RLS) | Restrict which rows a user can see |
| Audit | SQL Auditing | Track operations to Storage, Log Analytics, or Event Hubs |
| Threat detection | Microsoft Defender for SQL | ML-based anomaly detection: SQL injection, brute force |
| Integrity | Ledger tables | Tamper-evident cryptographic proof of data integrity |

## Authentication

### Best Practices
1. Use Microsoft Entra ID — single identity plane, conditional access, MFA, centralized audit
2. Avoid SQL authentication in production — passwords are weak, no MFA, no conditional access
3. Use managed identities for app-to-database connections — zero credential management
4. Entra groups for RBAC — manage access at group level, not individual user level
5. Regularly review access permissions — conduct access reviews, remove unnecessary privileges
6. Never concatenate user input into SQL — use parameterized queries to prevent injection

## Encryption

### TDE vs Always Encrypted

| Aspect | TDE | Always Encrypted |
|---|---|---|
| Scope | Entire database (files, logs, backups) | Specific columns |
| Encryption point | Server side (at rest only) | Client side (end-to-end) |
| DBA can see data? | Yes (decrypted in memory) | No (key only on client) |
| App changes needed | None | Minimal (driver changes) |
| Operations on encrypted data | Full | Equality comparison only |
| Enabled by default? | Yes (all new databases) | No (must configure) |

- Always Encrypted with Secure Enclaves adds pattern matching, sorting, in-place encryption
- TDE uses two-key hierarchy: AES-256 DEK encrypted by RSA 2048 master key
- Customer-managed keys via Azure Key Vault provide key rotation and access control

### Encryption in Transit
- TLS 1.2 enforced for all Azure database services
- Minimum TLS version configurable on SQL Database and PostgreSQL/MySQL

## Network Security
- Private endpoints: recommended for all production databases
- VNet service endpoints: legacy approach, private endpoints preferred
- Firewall rules: IP allowlist for specific access patterns
- No public access for production — disable public network access entirely

## Data Protection

### Dynamic Data Masking
- Masks sensitive data on-the-fly in query results — data in database is not changed
- Masking functions: default, email, random, custom string (e.g., XXX-XX-0000 for SSN)
- Auto-discovery identifies potentially sensitive data and recommends masking
- Excluded users can see unmasked data

### Row-Level Security
- Controls access at the row level based on user identity or execution context
- Filter predicates: hide rows the user should not see
- Block predicates: prevent writes to rows the user should not modify
- Implemented as security policy with inline table-valued functions

### Ledger Tables
- Tamper-evident logging with blockchain-style hash chains
- Two types: updatable ledger tables (track changes) and append-only (insert-only)
- Cryptographic proof of data integrity for regulatory compliance

## Auditing and Threat Detection
- SQL Auditing: track all database operations to Storage, Log Analytics, or Event Hubs
- Microsoft Defender for SQL: vulnerability assessment and threat detection
- Data Discovery and Classification: identify, classify, and label sensitive columns
- SQL Vulnerability Assessment: discover and remediate potential vulnerabilities

## Cosmos DB Security
- Entra ID authentication supported with RBAC for data plane
- Resource tokens for fine-grained per-container or per-partition access
- IP firewall rules, VNet service endpoints, private endpoints
- Always encrypted at rest (service-managed or customer-managed keys)
- Primary/secondary keys provide full access — treat as secrets, prefer Entra ID

## PostgreSQL Security
- Entra ID authentication supported on Flexible Server
- SSL/TLS enforced by default
- Private endpoints and VNet integration
- PostgreSQL native RBAC (roles, schemas, row-level security)
- Server-level firewall rules
