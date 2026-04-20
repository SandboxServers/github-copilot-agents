# Backup & Restore

## Azure SQL Database

### Automated Backups
- Full backups weekly, differential every 12-24 hours, transaction log every 5-10 minutes
- Backup storage redundancy: LRS, ZRS, or GRS (geo-redundant, default)
- Fully automatic — no user action required

### Point-in-Time Restore (PITR)
- Restore to any point within retention period (1-35 days, default 7)
- Creates a new database (cannot restore in-place)
- Restore time depends on database size and transaction log volume

### Long-Term Retention (LTR)
- Weekly, monthly, and/or yearly full backup copies up to 10 years
- Supports immutable storage for compliance (WORM — write once, read many)
- Use case: regulatory compliance requiring multi-year retention

### Geo-Restore
- Restore from geo-replicated backup to any Azure region
- RPO: up to 1 hour; RTO: hours (size-dependent)
- Available when GRS backup storage is configured
- Use case: DR when failover groups are not configured

### Hyperscale Backups
- Snapshot-based — near-instant regardless of database size
- Retention: 7 days default, configurable 1-35 days
- LTR available up to 10 years
- Restore creates new database from snapshots — significantly faster than traditional

## Cosmos DB

### Continuous Backup (Preferred)
- Point-in-time restore to any second within retention window
- 7-day retention (default, lower cost) or 30-day retention (higher cost)
- Restores to a new account (cannot restore in-place)
- Granularity: per-container or per-database
- Best for most production scenarios requiring fine-grained recovery

### Periodic Backup
- Full backup at configurable intervals (1-24 hours)
- Less granular — restore only to backup points, not arbitrary seconds
- Lower cost than continuous backup
- Use case: cost-sensitive workloads where fine-grained PITR is not needed

### Comparison

| Feature | Continuous | Periodic |
|---|---|---|
| Granularity | Any second | Backup interval only |
| Retention | 7 or 30 days | Configurable copies |
| Cost | Higher | Lower |
| Recommendation | Production | Dev/test, cost-sensitive |

## PostgreSQL Flexible Server
- Full + incremental backups with configurable retention (7-35 days)
- PITR to any point within retention window
- Geo-redundant backup available for cross-region restore
- On-demand backup before risky operations (schema changes, deployments)
- Restore always creates a new server instance

## MySQL Flexible Server
- Full + incremental backups, configurable retention (1-35 days, default 7)
- PITR within retention window
- Geo-redundant backup available
- Same restore model as PostgreSQL — creates new server instance

## Redis
- RDB snapshots: periodic point-in-time snapshots (Premium+ only)
- AOF persistence: logs every write operation (Premium+ only)
- Import/export for migration between caches
- For recovery scenarios, not primary data storage

## Best Practices
- Test restores regularly — untested backups are not backups
- Know your RPO and RTO for each database and tier
- Use LTR for compliance requirements
- Automate backup validation with periodic test restores
- Document restore procedures in runbooks
- Disable HA and read replicas before PostgreSQL/MySQL migration
