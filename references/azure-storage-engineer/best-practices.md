# Azure Storage Best Practices

Production-grade recommendations for Azure Storage security, performance, cost, and resilience.

## Authentication & Access Control

**Use managed identity for all application access**:
- Assign Storage Blob Data Reader/Contributor roles via RBAC to managed identities
- Disable shared key access at the account level (`allowSharedKeyAccess: false`)
- Eliminates key rotation burden and credential leakage risk
- SAS tokens only when external party access is required — use stored access policies with short expiry

**Follow least-privilege RBAC**:
- Prefer data-plane roles (e.g., Storage Blob Data Reader) over control-plane roles (e.g., Contributor)
- Scope assignments to resource group or storage account, not subscription
- Use Entra ID Conditional Access policies to restrict access by location or device compliance

## Network Security

**Private endpoints for all production storage accounts**:
- Create separate private endpoints for each service: blob, dfs, file, table, queue
- Configure private DNS zones for automatic name resolution
- Disable public network access after private endpoints are in place
- Copy operations require network access to both source and destination accounts

**Storage account firewall as defense in depth**:
- Start with "Deny" default and add exceptions
- Use service endpoints for same-region VNet traffic as a lower-cost alternative
- Avoid "Allow Azure services" — use resource instance rules or managed identity instead
- Regularly audit firewall rules for stale IP ranges

## Data Protection & Recovery

**Enable soft delete and versioning for all production data**:
- Blob soft delete: 14+ days retention
- Container soft delete: 7+ days retention
- Blob versioning for automatic point-in-time recovery
- Always pair versioning with lifecycle rules to auto-delete old versions

**Point-in-time restore for critical workloads**:
- Requires versioning + soft delete + change feed (all three)
- Restores block blobs to a prior state — useful for bulk corruption recovery
- Configure 1-7 day restore window based on RPO requirements

**Immutable storage for compliance**:
- Use time-based retention for regulatory data (SEC 17a-4, CFTC, FINRA)
- Lock policies only after thorough testing — locked retention cannot be shortened
- Legal holds for litigation or investigation — can be toggled without time commitment

## Encryption

**Customer-managed keys when compliance requires it**:
- Store keys in Azure Key Vault with soft delete and purge protection enabled
- Enable automatic key rotation in Key Vault
- Encryption scopes allow per-container or per-blob key assignment
- Microsoft-managed keys are sufficient for most workloads — CMK adds operational overhead

## Cost Optimization

**Lifecycle management from day one**:
- Transition blobs from Hot → Cool → Cold → Archive based on last access time
- Delete old versions and snapshots automatically (e.g., versions older than 30 days)
- Use access time tracking (enable last access time) for data-driven tier decisions
- Review lifecycle policy effectiveness monthly via Storage Analytics

**Right-size redundancy to workload requirements**:
- LRS: Dev/test and non-critical workloads only
- ZRS: Production workloads requiring zone resilience
- GRS/GZRS: Disaster recovery with cross-region replication
- RA-GRS/RA-GZRS: Read access to secondary for high-availability reads
- Test failover regularly if using GRS — do not assume it works untested

## Performance

**Choose the right account and tier**:
- Premium block blob accounts for latency-sensitive workloads (single-digit ms)
- Standard v2 for general-purpose storage with lifecycle management
- Premium file shares for Azure Files workloads requiring high IOPS
- Premium page blobs only for unmanaged VM disks (prefer Managed Disks)

**Optimize blob naming for throughput**:
- Use hash prefixes or GUIDs to distribute load across partitions
- Avoid sequential or timestamp-based prefixes (causes partition hotspots)
- For ADLS Gen2, design directory structure to match query patterns

## Data Migration

**Use the right tool for the scale**:
- AzCopy: Best for programmatic, scriptable bulk transfers (up to TBs)
- Azure Data Box: For 40+ TB offline migrations — ship physical device
- Storage Mover: For ongoing migrations with agent-based orchestration
- Azure File Sync: For hybrid scenarios keeping data in sync with on-prem

## Analytics Workloads

**Use ADLS Gen2 (hierarchical namespace) for analytics**:
- Atomic directory operations and POSIX-like ACLs
- Required for Synapse, Databricks, and HDInsight integration
- Design directory layout: `/{zone}/{subject}/{version}/{partition}`
- Separate storage accounts for raw, curated, and presentation zones

## Monitoring & Operations

**Enable comprehensive monitoring**:
- Storage Analytics logging for request-level diagnostics
- Azure Monitor metrics for capacity, transactions, latency, and availability
- Diagnostic settings to send logs to Log Analytics workspace
- Set alerts on availability drops, latency spikes, and throttling (E2E latency > 200ms)

**Tag and organize storage accounts**:
- Environment, cost center, owner, and data classification tags on every account
- Resource locks (CanNotDelete) on production storage accounts
- Azure Policy to enforce naming, tagging, and security standards

## Related References

- [security.md](security.md) — Detailed security configuration
- [lifecycle-management.md](lifecycle-management.md) — Tier transition rules
- [redundancy.md](redundancy.md) — Redundancy selection guide
- [anti-patterns.md](anti-patterns.md) — What to avoid
- [gotchas.md](gotchas.md) — Surprising behaviors and edge cases
