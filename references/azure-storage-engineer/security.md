# Storage Security

Comprehensive security model for Azure Storage covering authentication, authorization, encryption, network, and compliance.

## Authentication Methods

| Method | Security Level | Use Case |
|--------|---------------|----------|
| **Managed Identity + RBAC** | Best | Azure services, no credentials to manage |
| **Workload Identity Federation** | Excellent | CI/CD pipelines, no secrets |
| **Microsoft Entra ID + RBAC** | Good | Service principals with certificates |
| **Shared Key** | Poor | Legacy, disable when possible |
| **SAS Tokens** | Moderate | Scoped, time-limited external access |
| **Anonymous Access** | None | Disable by default, public content only |

## Authorization (RBAC Roles)

| Role | Permissions | Scope Guidance |
|------|-------------|---------------|
| **Storage Blob Data Owner** | Full access + manage ACLs | Account admins only |
| **Storage Blob Data Contributor** | Read/write/delete blobs | Application service accounts |
| **Storage Blob Data Reader** | Read-only access | Reporting, analytics consumers |
| **Storage Queue Data Contributor** | Read/write/delete queue messages | Queue-based applications |
| **Storage Table Data Contributor** | Read/write/delete table entities | Table-based applications |

**Best practice**: Assign at container level, not account level. Use least privilege.

## Encryption

**At rest**:
- Default: SSE with Microsoft-managed keys (AES-256, FIPS 140 validated)
- CMK via Key Vault: Your key wraps the storage encryption key
- CMK with HSM: Hardware security module-backed keys
- Key Vault requires soft delete + purge protection enabled
- Auto-rotation: Omit key version → Azure checks daily for new version

**In transit**:
- HTTPS enforced (require secure transfer setting)
- TLS 1.2 minimum (disable older versions)
- SMB 3.x encryption for Azure Files

## Network Security

| Method | Description | Recommendation |
|--------|-------------|----------------|
| **Private endpoints** | Private IP from VNet, traffic on Microsoft backbone | Recommended for production |
| **Service endpoints** | Route traffic through VNet (still public IP) | Alternative, less secure |
| **Firewall rules** | IP allow list for public access | Complement to private endpoints |
| **VNet rules** | Restrict to specific virtual networks | Use with service endpoints |

**Private endpoint details**:
- One per sub-service (blob, file, table, queue, dfs, web)
- DNS: Private DNS zone `privatelink.blob.core.windows.net`
- Can disable public endpoint entirely (`publicNetworkAccess: Disabled`)

## Immutable Storage (WORM)

| Type | Duration | Use Case |
|------|----------|----------|
| **Time-based retention** | Configurable period | SEC 17a-4, CFTC 1.31 compliance |
| **Legal hold** | Indefinite (until removed) | Litigation, investigations |
| **Version-level** | Per blob version | Granular control |
| **Container-level** | Entire container | Broad compliance |

- Locked policies cannot be shortened (irreversible)
- Unlocked policies can be modified
- Compliance: SEC, FINRA, CFTC, HIPAA

## Threat Protection

- **Microsoft Defender for Storage**: Anomaly detection on access patterns
- **Sensitive data scanning**: Detect PII, credentials, secrets in blobs
- **Alerts**: Suspicious access, unusual data exfiltration, anonymous access
- **Integration**: Microsoft Sentinel, Azure Monitor, Security Center

## SAS Token Guidance

**When SAS tokens are appropriate**:
- Time-limited access for external parties
- Pre-signed upload URLs for client-side uploads
- Integration with systems that can't use Entra ID

**SAS best practices**:
- HTTPS only, shortest viable expiration
- Minimum permissions (read-only if possible)
- Use stored access policies for revocation capability
- Service SAS (not account SAS) for scoped access

## Best Practices

1. **Disable shared key access** on all production accounts
2. **Disable anonymous access** at the account level
3. **Use private endpoints** for production workloads
4. **Enable Defender for Storage** for threat detection
5. **Use immutable storage** for compliance data
6. **Rotate SAS tokens** regularly, use stored access policies
7. **CMK for sensitive data** with auto-rotation enabled
8. **Require secure transfer** (HTTPS) on all accounts
9. **Apply resource locks** to prevent accidental deletion
10. **Audit access** via Azure Monitor diagnostic logs

## Related References

- [redundancy.md](redundancy.md) — Data protection through replication
- [data-protection.md](data-protection.md) — Soft delete, versioning, backups
- [anti-patterns.md](anti-patterns.md) — Common security mistakes
