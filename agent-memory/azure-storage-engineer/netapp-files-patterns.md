# Azure NetApp Files Patterns

Deployment patterns, sizing guidance, and operational considerations for Azure NetApp Files.

## When to Choose Azure NetApp Files

| Requirement | Azure Files Premium | Azure NetApp Files |
|-------------|--------------------|--------------------|
| Sub-millisecond latency | No (single-digit ms) | Yes (< 1ms) |
| NFS + SMB on same volume | No (one protocol per share) | Yes (dual-protocol) |
| Throughput > 10 Gbps | Limited | Up to 4,500 MiB/s per volume |
| Oracle/SAP HANA workloads | Not certified | SAP HANA and Oracle certified |
| POSIX-compliant file system | Limited | Full POSIX compliance |
| Protocol version flexibility | NFS 4.1 only | NFS 3.0, 4.1, SMB 3.x |

**Choose Azure NetApp Files when**: enterprise NAS migration, SAP/Oracle workloads, HPC, or sub-ms latency is required.

## Service Tiers and Sizing

| Service Level | Throughput per TiB | Best For |
|---------------|-------------------|----------|
| **Standard** | 16 MiB/s | Archive, backup, infrequently accessed data |
| **Premium** | 64 MiB/s | General-purpose file shares, databases, dev/test |
| **Ultra** | 128 MiB/s | Latency-sensitive, high-throughput workloads |

**Capacity pool model**:
- Minimum capacity pool: 1 TiB (billed even if underutilized)
- Volumes consume capacity from the pool — multiple volumes share a pool
- Dynamic service level change: move volumes between pools without downtime
- Cool access option: auto-tier cold data to reduce costs (Standard and Premium tiers)

## Replication Patterns

**Cross-region replication** (disaster recovery):
- Async replication to a paired or supported non-standard region pair
- Schedules: every 10 minutes, hourly, or daily
- Destination volume is read-only during normal operation — becomes read-write on failover
- Failover is manual — you must break replication peering and mount destination
- Cost: billed per GiB replicated, plus destination volume capacity charges
- Destination can use a lower service level to reduce cost

**Cross-zone replication** (business continuity):
- Same as cross-region but within a single region across availability zones
- Lower latency replication compared to cross-region
- Source volume must be in an availability zone

**Cross-zone-region replication** (combined):
- Single source volume can replicate to both a different zone AND a different region
- Provides both business continuity and disaster recovery from one source
- Supports dual cross-region or dual cross-zone destination combinations

## Snapshot and Backup Patterns

**Snapshots**:
- Near-instant, space-efficient (only changed blocks stored)
- Maximum 255 snapshots per volume
- Snapshot policies: automated schedules (hourly, daily, weekly, monthly)
- Users can self-service restore from `.snapshot` directory (NFS) or `~snapshot` (SMB)

**Azure NetApp Files backup**:
- Vault-based backup to Azure storage (independent of volume lifecycle)
- Policy-driven: daily and weekly schedules with retention rules
- Restores create new volumes from backup — not in-place restore
- Backup is async and does not impact volume performance

## Networking Considerations

- Volumes are deployed into a **delegated subnet** — one subnet per NetApp account per region
- Subnet must be `/28` or larger (recommended `/26` for growth)
- Supports Standard network features (NSG, UDR, private endpoints) on new volumes
- For on-premises access: ExpressRoute or VPN Gateway to the VNet

## Operational Patterns

**Dynamic pool management**:
- Move volumes between capacity pools to change service level on the fly
- Useful for workloads with predictable peak hours — scale up during business hours, down overnight
- Pool resizing is online and immediate

**Monitoring and alerts**:
- Azure Monitor metrics: throughput, IOPS, latency per volume
- Capacity alerts: set threshold alerts at 80% pool utilization
- Replication lag monitoring: alert if RPO exceeds expected threshold

**Ransomware protection** (preview):
- Monitors for suspicious activity via file extension profiling and entropy patterns
- Auto-creates snapshots on threat detection for rapid recovery
- Attack reports retained for 30 days

## Related References

- [azure-files-netapp.md](azure-files-netapp.md) — Azure Files comparison and File Sync
- [redundancy.md](redundancy.md) — Storage redundancy strategies
- [performance.md](performance.md) — Performance optimization patterns
