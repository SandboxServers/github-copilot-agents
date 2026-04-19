# Replication & Disaster Recovery

## Azure SQL Database

### Geo-Replication Options
| Feature | Active Geo-Replication | Failover Groups |
|---|---|---|
| Continuous sync | Yes | Yes |
| Multiple secondaries | Up to 4 | 1 secondary region |
| Same region secondary | Yes | No (must be different region) |
| Fail over multiple DBs | No (individual DB) | Yes (group of DBs as unit) |
| Connection string unchanged | No (must update app) | Yes (listener endpoints auto-redirect) |
| Read-scale | Yes | Yes |
| License-free standby | Yes (if DR-only) | Yes (if DR-only) |

- **Failover groups**: preferred for most scenarios — listener endpoints mean no connection string changes after failover. Fail over all databases in the group as a unit.
- **Active geo-replication**: when you need multiple secondaries, same-region secondary, or more granular control (individual database level).
- **License-free standby replica**: if secondary is DR-only (no read workload), save on SQL Server licensing costs.

### RPO/RTO Targets
| Strategy | RPO | RTO |
|---|---|---|
| PITR (point-in-time restore) | ~5-10 minutes (log backup interval) | Minutes to hours (size dependent) |
| Geo-restore | Up to 1 hour | Hours |
| Active geo-replication | < 5 seconds | Seconds to minutes |
| Failover groups | < 5 seconds | Seconds to minutes |
| LTR restore | Weekly/monthly/yearly (policy-dependent) | Hours |

### Hyperscale HA
- 0-4 configurable secondary replicas (named replicas)
- Zone-redundant HA supported
- Multi-AZ available across all serverless and provisioned sizes
- Read scale-out via named replicas
- Backup retention: 7 days default for serverless (configurable up to 35 days for provisioned)

## Cosmos DB Replication

### Multi-Region Architecture
- **Multi-region writes** (active-active): automatic, configure conflict resolution policies (Last Writer Wins by default, custom merge procedures, or async)
- **Single-region write + multi-region read**: automatic failover to promoted read region
- Service-managed failover (automatic) or manual failover — configurable per account
- Each physical partition is part of a **partition-set** — a geo-distributed overlay of physical partitions across all configured regions
- Two-level nested consensus protocol: replica-set level (within physical partition) + partition-set level (cross-region ordering)

### RPO by Consistency Level (Verified via MS Learn)
| Consistency | Single Write Region | Multi-Write Region |
|---|---|---|
| Strong | RPO = 0 | Not supported with multi-write |
| Bounded Staleness | RPO = K versions / T time | RPO = K versions / T time |
| Session/Prefix/Eventual | < 15 minutes | < 15 minutes |

- K/T minimums: single-region = 10 writes or 5 seconds; multi-region = 100,000 writes or 300 seconds
- Strong consistency in multi-region: falls back to single-write-region with Bounded Staleness behavior

## PostgreSQL Flexible Server

### HA Options
- **Zone-redundant HA**: synchronous replication to standby in different AZ, automatic failover (60-120 seconds)
- **Same-zone HA**: synchronous replication within same AZ (protects against node failure, not zone failure)
- Failover: planned = zero data loss; unplanned = potential minimal data loss

### Read Replicas
- Async replication to up to 5 replicas (same or different region)
- Cross-region replicas for geo-read distribution and DR
- Replica can be promoted to standalone server
- **Gotcha**: async = replicas may lag. Not suitable for read-your-writes patterns.

### Geo-Redundant Backup
- Available for cross-region restore from backup
- Not the same as real-time replication — recovery from backup means higher RPO
