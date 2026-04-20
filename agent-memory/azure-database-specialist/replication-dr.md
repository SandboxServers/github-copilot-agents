# Replication & Disaster Recovery

## Azure SQL Database

### Geo-Replication Options

| Feature | Active Geo-Replication | Failover Groups |
|---|---|---|
| Multiple secondaries | Up to 4 | 1 secondary region |
| Same region secondary | Yes | No (must be different region) |
| Fail over multiple DBs | No (individual DB) | Yes (group of DBs as unit) |
| Connection string unchanged | No (must update app) | Yes (listener endpoints auto-redirect) |
| License-free standby | Yes (if DR-only) | Yes (if DR-only) |

- Failover groups preferred for most scenarios — listener endpoints mean no connection string changes
- Active geo-replication for multiple secondaries or more granular per-database control

### Zone-Redundant Deployment
- Available for General Purpose, Business Critical, and Hyperscale tiers
- Synchronous replication across availability zones within a region
- 99.995% SLA with zone redundancy on Business Critical

### RPO/RTO Targets

| Strategy | RPO | RTO |
|---|---|---|
| PITR (point-in-time restore) | ~5-10 min | Minutes to hours |
| Geo-restore | Up to 1 hour | Hours |
| Active geo-replication | < 5 seconds | Seconds to minutes |
| Failover groups | < 5 seconds | Seconds to minutes |

## Cosmos DB

### Multi-Region Architecture
- Multi-region writes (active-active): automatic conflict resolution (Last Writer Wins default)
- Single-region write + multi-region read: automatic failover to promoted read region
- Service-managed failover (automatic) or manual — configurable per account
- Each physical partition is part of a partition-set across all configured regions

### RPO by Consistency Level

| Consistency | Single Write Region | Multi-Write Region |
|---|---|---|
| Strong | RPO = 0 | Not supported with multi-write |
| Bounded Staleness | RPO = K versions / T time | RPO = K versions / T time |
| Session/Prefix/Eventual | < 15 minutes | < 15 minutes |

## PostgreSQL Flexible Server

### HA Options
- Zone-redundant HA: synchronous replication to standby in different AZ, 60-120s failover
- Same-zone HA: synchronous replication within same AZ (node protection only)
- Planned failover: zero data loss; unplanned: potential minimal data loss

### Read Replicas
- Async replication to up to 5 replicas (same or different region)
- Cross-region replicas for geo-read distribution and DR
- Replica promotes to standalone server on failover
- Async means replicas may lag — not suitable for read-your-writes

### Geo-Redundant Backup
- Cross-region restore from backup — higher RPO than real-time replication
- Available for PostgreSQL and MySQL Flexible Server

## MySQL Flexible Server
- Same HA model as PostgreSQL: zone-redundant and same-zone options
- Read replicas up to 5, any region, async replication
- Manual failover by promoting replica to standalone server

## Redis
- Geo-replication: Premium tier supports passive geo-replication (async)
- Enterprise tier supports active geo-replication (active-active across regions)
- Data persistence (RDB/AOF) for recovery from full restart

## Failover Testing Best Practices
- Conduct regular DR drills at least quarterly
- Test failover groups with actual application connections
- Measure actual RTO and compare to targets
- Document runbooks with step-by-step failover procedures
- Monitor replication lag on all async replicas
- Validate data integrity after every failover test
- Test both planned and unplanned failover scenarios
- Include application-level validation in DR tests (not just database connectivity)
