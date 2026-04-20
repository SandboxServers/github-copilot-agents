# PostgreSQL & MySQL Flexible Server

## Azure Database for PostgreSQL Flexible Server

### Compute Tiers

| Tier | Series | Best For |
|---|---|---|
| Burstable | B-series | Dev/test, low-traffic apps |
| General Purpose | Dv5 | Most production workloads |
| Memory Optimized | Edsv5 | High-memory analytics, caching |

### High Availability
- Zone-redundant HA: synchronous standby in different AZ, 99.99% SLA, 60-120s failover
- Same-zone HA: synchronous standby in same AZ, 99.95% SLA, 60-120s failover
- HA doubles compute cost (standby replica)
- Planned failover: zero data loss; unplanned: potential minimal data loss

### Key Features
- Read replicas: up to 5, any region, asynchronous replication
- Point-in-time restore: 7-35 days configurable retention
- Built-in PgBouncer connection pooling for high-connection workloads
- Stop/start capability for dev/test cost savings
- Custom maintenance windows

### Extensions
- PostGIS: geospatial data and queries
- pgvector: vector similarity search for AI embeddings (HNSW and IVF indexes)
- pg_cron: scheduled jobs within the database
- pg_stat_statements: query performance tracking
- TimescaleDB: time-series data optimization
- Extensions must be allowlisted via azure.extensions parameter before use

### When PostgreSQL over SQL Database
- Open-source requirement or vendor lock-in concerns
- PostGIS needed for geospatial workloads
- pgvector needed for AI embedding search
- JSON-heavy workloads (JSONB is excellent for semi-structured data)
- Cost sensitivity — often cheaper than equivalent SQL Database tiers
- Need built-in connection pooling (PgBouncer) for microservices
- Team has PostgreSQL expertise

## Azure Database for MySQL Flexible Server

### Compute Tiers
- Same tier structure as PostgreSQL: Burstable, General Purpose, Memory Optimized
- Supports MySQL 5.7 and 8.0

### Key Features
- Zone-redundant HA and same-zone HA (same model as PostgreSQL)
- Read replicas for scale-out reads
- Data-in replication for migration and hybrid scenarios
- Stop/start for dev/test cost savings

### When MySQL
- WordPress, Drupal, Magento, or other CMS platforms
- LAMP/LEMP stack migration from on-prem or other clouds
- Team expertise is MySQL-specific
- Migrating from Amazon RDS MySQL, Aurora MySQL, or Google Cloud SQL MySQL
- Specific application requirements mandating MySQL compatibility

### MySQL vs PostgreSQL Decision

| Factor | PostgreSQL | MySQL |
|---|---|---|
| Extension ecosystem | Richer (PostGIS, pgvector, TimescaleDB) | More limited |
| JSON support | JSONB (binary, indexable, excellent) | JSON (text-based, improving) |
| Connection pooling | Built-in PgBouncer | External (ProxySQL) |
| Replication | Logical + physical | Binary log + GTID |
| Best for | Complex queries, GIS, AI/vector, analytics | Web apps, CMS, LAMP stacks |
| Community | Enterprise/academic/analytics strong | Web development strong |
