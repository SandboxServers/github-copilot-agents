## Security Deep Dive

### Authentication Hierarchy (Best to Worst)

```
1. Managed Identity + RBAC          ← BEST: No credentials, no rotation, Azure-managed
2. Workload Identity Federation     ← For CI/CD: No secrets, federated trust
3. Microsoft Entra ID + RBAC       ← Service principals with certificates (not secrets)
4. Storage account keys             ← AVOID: Full access, hard to rotate, shared
5. SAS tokens (service/account)     ← AVOID IF POSSIBLE: Time-limited but leakable
6. Anonymous/public access          ← NEVER in production
```

### Why SAS Tokens Are Usually Wrong
- **They leak**: Embedded in URLs, logged in proxies, shared in emails, committed to repos
- **They're hard to revoke**: Account SAS requires rotating storage keys (breaks ALL SAS tokens); Service SAS with stored access policy is slightly better
- **They grant too much**: Often created with overly broad permissions
- **They expire badly**: Too short = user disruption; too long = security risk
- **Better alternatives exist**: Managed identity for Azure services, Entra ID for users, workload identity federation for CI/CD

### When SAS Tokens Are Actually Appropriate
- Time-limited access for external parties (with service SAS + stored access policy)
- Pre-signed upload URLs for client-side uploads
- Integration with systems that can't use Entra ID
- Always: HTTPS only, shortest viable expiration, minimum permissions, stored access policy for revocation

### Private Endpoints
- Assign private IP from VNet to storage account
- Traffic stays on Microsoft backbone (never hits public internet)
- Can disable public endpoint entirely (`publicNetworkAccess: Disabled`)
- DNS: Private DNS zone `privatelink.blob.core.windows.net` resolves to private IP
- One private endpoint per sub-service (blob, file, table, queue, dfs, web)

### Customer-Managed Keys (CMK)
- Default: Microsoft-managed keys (AES-256, FIPS 140 validated)
- CMK: Your key in Key Vault wraps the storage encryption key
- Key Vault must have soft delete + purge protection enabled
- Support RSA 2048, 3072, 4096
- Auto-rotation: Omit key version → Azure checks daily for new version
- Blob and Files always protected by CMK; Queue and Table require opt-in at account creation
- Managed identity (user-assigned or system-assigned) authenticates to Key Vault

### Immutable Storage
- **Time-based retention**: WORM — blob can't be modified or deleted until retention expires
- **Legal hold**: Indefinite — blob can't be modified or deleted until hold is removed
- **Version-level immutability**: Policy on individual versions (more granular)
- **Container-level immutability**: Policy on entire container
- **Locked vs Unlocked policies**: Locked = cannot shorten retention (irreversible); Unlocked = can modify
- Use case: Financial regulations (SEC 17a-4, CFTC 1.31), healthcare (HIPAA), legal hold
