# Storage Anti-Patterns

Common mistakes and their corrections when working with Azure Storage.

## Anti-Pattern Reference

| Anti-Pattern | Problem | Better Approach |
|-------------|---------|----------------|
| **Everything in Hot tier** | Overspend on storage costs | Lifecycle management, access time tracking |
| **Sequential blob names** | Partition hotspot, throttling | Hash prefix or GUID prefix |
| **Shared Key everywhere** | Security risk, no audit trail | Entra ID RBAC, disable shared key |
| **No soft delete** | Accidental data loss, no recovery | Enable blob and container soft delete |
| **Public anonymous access** | Data exposure risk | Disable at account level, use SAS or RBAC |
| **Single storage account for everything** | Blast radius, limits, management | Separate accounts by workload/environment |
| **GRS without testing failover** | False confidence in DR | Test customer-initiated failover regularly |
| **SAS tokens with long expiry** | Token compromise window | Short-lived SAS, stored access policies |
| **No lifecycle management** | Growing costs, stale data | Implement lifecycle rules from day 1 |
| **Archive tier without rehydration plan** | Can't access data when needed | Document rehydration SLA, test process |

## Expanded Anti-Pattern Details

**Versioning without lifecycle rules**:
- Every write creates a new version → capacity grows unbounded
- Costs skyrocket, often unnoticed for months
- Fix: Always add version cleanup rules when enabling versioning

**Archiving small files individually**:
- Transaction cost to archive + rehydrate exceeds storage savings
- Fix: Tar/zip small files before archiving, or leave in Cool/Cold tier

**Ignoring soft delete in cost estimates**:
- Deleted data still costs money during retention period
- Fix: Factor retention capacity into budget planning

**LRS for production workloads**:
- Single-zone failure loses all copies
- Fix: ZRS minimum for any production data

**Not using private endpoints**:
- Data traverses public internet even within Azure
- Fix: Private endpoints for all production storage accounts

## Questions This Specialist Always Asks

### Scoping
- What type of data? (Objects, files, blocks, structured?)
- How much data today? Growth rate?
- Access patterns? (Frequency, read/write ratio, hot vs cold split)
- Latency requirements? (Sub-ms → NetApp/Premium; seconds → Standard)

### Security
- Who needs access? (Users, applications, external parties, CI/CD?)
- How is access authenticated today? (Keys, SAS, managed identity?)
- Compliance requirements? (Immutability, CMK, data residency?)
- Network access? (Public internet, VNet only, on-prem hybrid?)

### Cost
- Current monthly storage spend?
- Are lifecycle policies in place?
- Versioning enabled? With cleanup rules?
- How many storage accounts? (Too few → limits; too many → overhead)

### Resilience
- RPO / RTO requirements?
- Can data be regenerated from source?
- Multi-region requirements?
- Backup strategy? (Azure Backup, snapshots, cross-account replication?)

## Related References

- [lifecycle-management.md](lifecycle-management.md) — Lifecycle rules to prevent cost growth
- [security.md](security.md) — Proper authentication and access
- [redundancy.md](redundancy.md) — Choosing the right redundancy
- [blob-access-tiers.md](blob-access-tiers.md) — Tier optimization
