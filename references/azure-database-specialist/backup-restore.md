# Backup & Restore

## Azure SQL Database

### Automated Backups
- **Full backups**: weekly
- **Differential backups**: every 12-24 hours
- **Transaction log backups**: every 5-10 minutes
- Backup storage redundancy options: LRS (locally redundant), ZRS (zone-redundant), GRS (geo-redundant, default)
- Backups are automatic — no user action required

### Point-in-Time Restore (PITR)
- Restore to any point within retention period
- Retention: 1-35 days (default 7 days)
- Creates a new database (cannot restore in-place)
- Restore time depends on database size and transaction log volume
- Available for all service tiers

### Long-Term Retention (LTR)
- Weekly, monthly, and/or yearly full backup copies
- Retention up to 10 years
- Supports immutable storage for compliance (WORM — write once, read many)
- LTR backups stored in Azure Blob Storage (separate from automated backups)
- **Use case**: regulatory compliance requiring multi-year retention

### Geo-Restore
- Restore from geo-replicated backup to any Azure region
- RPO: up to 1 hour (geo-replication lag of backup storage)
- RTO: hours (depends on database size)
- Available when GRS backup storage is configured
- **Use case**: DR when active geo-replication or failover groups are not configured

### Hyperscale Backup Specifics
- Snapshot-based backups — near-instant regardless of database size
- Backup storage retention: 7 days default (configurable 1-35 days)
- Backup storage options: LRS, ZRS, GRS
- LTR available up to 10 years
- Restore creates new database from snapshots — significantly faster than traditional restore

## Cosmos DB

### Continuous Backup (Preferred)
- Point-in-time restore to any second within retention window
- Retention tiers:
  - **7-day** retention (default, lower cost)
  - **30-day** retention (higher cost, more flexibility)
- Restores to a new account (cannot restore in-place)
- Granularity: per-container or per-database
- **Best for**: most production scenarios requiring fine-grained recovery

### Periodic Backup
- Full backup at configurable intervals (1-24 hours)
- Configurable number of backup copies retained
- Less granular than continuous — can only restore to a backup point, not arbitrary second
- **Use case**: cost-sensitive workloads where fine-grained PITR isn't needed

### Backup Comparison
| Feature | Continuous | Periodic |
|---|---|---|
| Granularity | Any second within window | Backup interval only |
| Retention | 7 or 30 days | Configurable copies |
| Restore target | New account | New account |
| Cost | Higher | Lower |
| Recommendation | Production | Cost-sensitive / dev-test |

## PostgreSQL Flexible Server

### Automated Backups
- Full + incremental backups
- Configurable retention: 7-35 days
- PITR: restore to any point within retention window (to the second)
- Geo-redundant backup available for cross-region restore

### On-Demand Backup
- Trigger manual backup before risky operations (schema changes, major deployments)
- Stored alongside automated backups

### Key Considerations
- Disable HA and read replicas before migration — enable only after migration completes and database is stable
- Geo-redundant backup provides cross-region restore but with higher RPO than real-time replication
- Restore always creates a new server instance

## MySQL Flexible Server

### Automated Backups
- Full + incremental, configurable retention (1-35 days, default 7)
- PITR within retention window
- Geo-redundant backup available
- Same restore model as PostgreSQL — creates new server instance
