# Database Migration

## Azure Database Migration Service (DMS)

### Supported Scenarios

#### Offline Migration (GA)

| Source | Target |
|---|---|
| SQL Server | Azure SQL Database / MI / VM |
| Amazon RDS SQL Server | Azure SQL DB / MI / VM |
| MySQL / RDS MySQL / Aurora MySQL / Cloud SQL MySQL | MySQL Flexible Server |
| MongoDB | Cosmos DB |
| PostgreSQL / RDS PostgreSQL | PostgreSQL Flexible Server |

#### Online Migration (Minimal Downtime — GA)

| Source | Target |
|---|---|
| SQL Server | Azure SQL MI / VM (NOT SQL Database) |
| Amazon RDS SQL Server | Azure SQL MI / VM |
| MySQL / RDS MySQL / Aurora MySQL / Cloud SQL MySQL | MySQL Flexible Server |
| MongoDB | Cosmos DB |
| PostgreSQL / RDS PostgreSQL | PostgreSQL Flexible Server |

Notable gap: online migration to Azure SQL Database is NOT supported — only offline. Online is supported for MI and VM targets.

### Offline vs Online Decision

| Factor | Offline | Online |
|---|---|---|
| Downtime | Full during migration | Minimal at cutover only |
| Complexity | Simple, fewer failure modes | More complex, replication involved |
| Restrictions | None significant | PostgreSQL requires primary keys on all tables |
| Best for | Planned maintenance windows | Large databases, business continuity critical |

## SQL Server to Azure SQL Migration

### Workflow
1. Assessment: Azure Migrate or Data Migration Assistant — compatibility check, feature parity
2. SKU recommendation: right-size target based on current workload metrics
3. Schema migration: DMA or SSDT for schema comparison and migration
4. Data migration: DMS (online or offline), BACPAC export/import, or transactional replication
5. Cutover: switch connection strings, verify data integrity, decommission source
6. Post-migration: enable HA, monitoring, backup policies; validate performance

### Method Comparison

| Method | Downtime | Best For |
|---|---|---|
| DMS offline | Full | Simple migrations, small databases |
| DMS online (MI/VM only) | Minimal | Large databases needing minimal downtime |
| BACPAC export/import | Full | Small databases, schema + data together |
| Transactional replication | Minimal | Complex scenarios, selective table migration |

## PostgreSQL Migration

### Built-in Migration Service
- Integrated directly in PostgreSQL Flexible Server (portal + CLI)
- Sources: on-prem PostgreSQL, Amazon RDS, Google Cloud SQL, Azure Single Server
- Online and offline modes available

### Workflow
1. Enable supported extensions on target (allowlist + CREATE EXTENSION)
2. Match server parameters between source and target
3. Migrate users and roles manually via pg_dumpall --globals-only
4. Remove superuser privileges (not supported in Flexible Server)
5. Disable HA and read replicas on target before migration
6. Run migration (online or offline)
7. Validate, then enable HA and replicas post-migration

### Key Constraints
- Online mode requires primary keys on all tables for replication tracking
- Superuser roles must be removed or adjusted before migration
- Extensions must be allowlisted and created on target before migration starts

## MySQL Migration
- DMS supports: MySQL, Amazon RDS MySQL, Aurora MySQL, Cloud SQL MySQL, Percona
- Both online and offline modes — all GA
- Data-in replication available for hybrid scenarios

## Cosmos DB Migration
- DMS for MongoDB to Cosmos DB (MongoDB API) — online and offline, both GA
- Data migration tool for JSON to Cosmos DB NoSQL
- Spark connector for bulk migration from any Spark-compatible source
- Design partition key strategy BEFORE migrating — cannot change after container creation

## Migration Checklist
1. Assess compatibility and fix blockers
2. Right-size target SKU from workload metrics
3. Test migration in dev environment first
4. Migrate with DMS (online or offline)
5. Validate data integrity and row counts
6. Run performance tests against target
7. Cutover with connection string switch
8. Enable HA, monitoring, and backup policies
9. Monitor for 2-4 weeks post-migration
