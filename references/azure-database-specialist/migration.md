# Database Migration

## Azure Database Migration Service (DMS)

### Supported Scenarios (Verified via MS Learn)

#### Offline Migration (GA)
| Source | Target | Status |
|---|---|---|
| SQL Server | Azure SQL Database | GA |
| SQL Server | Azure SQL Managed Instance | GA |
| SQL Server | SQL Server on Azure VM | GA |
| Amazon RDS SQL Server | Azure SQL DB / MI / VM | GA |
| Oracle | Azure SQL (via SSMA) | Preview |
| MySQL / RDS MySQL / Aurora MySQL / Cloud SQL MySQL / Percona | Azure Database for MySQL Flexible | GA |
| MongoDB | Azure Cosmos DB | GA |
| Azure Database for MySQL | Azure Database for MySQL Flexible | GA |

#### Online Migration (Minimal Downtime — GA)
| Source | Target | Status |
|---|---|---|
| SQL Server | Azure SQL Managed Instance | GA |
| SQL Server | SQL Server on Azure VM | GA |
| Amazon RDS SQL Server | Azure SQL MI / VM | GA |
| MySQL / RDS MySQL / Aurora MySQL / Cloud SQL MySQL / Percona | Azure Database for MySQL Flexible | GA |
| MongoDB | Azure Cosmos DB | GA |
| PostgreSQL / RDS PostgreSQL | Azure Database for PostgreSQL Flexible | GA |
| Azure Database for MySQL | Azure Database for MySQL Flexible | GA |

**Notable gap**: Online migration to Azure SQL Database is NOT supported — only offline. Online is supported for Managed Instance and Azure VM targets.

### Offline vs Online Decision (Verified via MS Learn)
| Factor | Offline | Online |
|---|---|---|
| Downtime | Full downtime during migration | Minimal downtime at cutover only |
| Complexity | Simple, fewer failure modes | More complex, replication involved |
| Restrictions | None significant | PostgreSQL online requires primary keys on all tables |
| Success rate | Higher (simpler process) | More failure modes due to replication complexity |
| Best for | Planned maintenance windows, smaller DBs | Large databases, business continuity critical |

## SQL Server to Azure SQL Migration

### Migration Workflow
1. **Assessment**: Azure Migrate or Data Migration Assistant (DMA) — compatibility check, feature parity analysis
2. **SKU recommendation**: right-size the target based on current workload metrics (CPU, IO, memory baselines)
3. **Schema migration**: DMA or SSDT for schema comparison and migration
4. **Data migration**: DMS (online or offline), BACPAC export/import, or transactional replication
5. **Cutover**: switch connection strings, verify data integrity, decommission source
6. **Post-migration**: enable HA, monitoring, backup policies; validate performance baselines

### Migration Methods Comparison
| Method | Downtime | Complexity | Best For |
|---|---|---|---|
| DMS offline | Full | Low | Simple migrations, small databases |
| DMS online (MI/VM only) | Minimal | Medium | Large databases needing minimal downtime |
| BACPAC export/import | Full | Low | Small databases, schema + data together |
| Transactional replication | Minimal | High | Complex scenarios, selective table migration |

## PostgreSQL Migration (Verified via MS Learn)

### Built-in Migration Service
- Integrated directly in Azure Database for PostgreSQL Flexible Server (Azure portal + CLI)
- Supports sources: on-prem PostgreSQL, Amazon RDS, Google Cloud SQL, Azure Single Server, other Flexible Servers
- Online and offline modes available

### Migration Workflow
1. Enable supported extensions on target (allowlist + `CREATE EXTENSION`)
2. Check `shared_preload_libraries` — some extensions require preload (restart needed)
3. Match server parameters manually between source and target
4. Migrate users and roles manually via `pg_dumpall --globals-only`
5. Remove superuser privileges (not supported in Flexible Server)
6. Disable HA and read replicas on target before migration
7. Run migration (online or offline)
8. Validate, then enable HA and replicas post-migration

### Key Constraints
- **Online mode requires primary keys** on all tables being migrated (for replication tracking)
- Superuser roles must be removed/adjusted before migration
- Extensions must be allowlisted and created on target before migration starts
- Some extensions need `shared_preload_libraries` configuration (requires restart)

## MySQL Migration
- DMS supports: MySQL, Amazon RDS MySQL, Aurora MySQL, Google Cloud SQL MySQL, Percona MySQL
- Both online and offline modes — all GA
- Schema migration supported via DMS
- Data-in replication available for hybrid scenarios with MyDumper/MyLoader

## Cosmos DB Migration
- DMS for MongoDB → Cosmos DB (MongoDB API) — online and offline, both GA
- Data migration tool for JSON → Cosmos DB NoSQL
- Spark connector for bulk migration from any Spark-compatible source
- **Critical**: design partition key strategy BEFORE migrating — partition key cannot be changed after container creation
- For MongoDB vCore: standard MongoDB tools (mongodump/mongorestore, mongoimport) work directly
