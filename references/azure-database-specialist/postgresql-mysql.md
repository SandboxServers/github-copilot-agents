# PostgreSQL & MySQL Flexible Server

## Azure Database for PostgreSQL Flexible Server

### Why PostgreSQL on Azure
- Full PostgreSQL engine (versions 13-17+), community extensions support
- Built-in pgBouncer connection pooling (critical for high-connection workloads — thousands of connections with low overhead)
- Stop/start capability for cost savings in dev/test
- Configurable server parameters (not locked down like some managed services)
- Custom maintenance windows — schedule maintenance for specific day/time
- Vector Search + Azure AI Extension for AI-powered applications
- Support for open-source extensions: PostGIS, pgvector, TimescaleDB, pg_trgm, hstore, and many more
- Extensions must be allowlisted before use — add to `azure.extensions` parameter, verify with `SHOW azure.extensions;`

### pgvector Extension (Verified via MS Learn)
- Adds open-source vector similarity search to PostgreSQL
- Extension binary name is `vector` (not pgvector) — use `CREATE EXTENSION vector;` to enable
- Must be added to allowlist first, then created per-database
- Supports HNSW and IVF index types for approximate nearest neighbor search
- Enables AI/ML embeddings storage and similarity search directly in PostgreSQL
- Avoids need for separate vector database — query vectors alongside relational data

### High Availability Options
| Configuration | SLA | Protection | Failover Time |
|---|---|---|---|
| **No HA** | ~99.9% | Node-level (auto-restart, relocate) | Minutes to hours |
| **Same-zone (Zonal)** | 99.95% | Node-level, synchronous standby in same AZ | 60-120 seconds |
| **Zone-redundant** | 99.99% | Zone-level, synchronous standby in different AZ | 60-120 seconds |

- HA doubles compute cost (standby replica)
- Zone-redundant protects against full AZ failure
- Planned failover: zero data loss, ~60-120 seconds of write blocking
- **Must disable HA and read replicas before migration** — enable only after migration completes
- **Gotcha**: zone-redundant not available in all regions

### When to Pick PostgreSQL over SQL Database
- Application already uses PostgreSQL (migration friction is real)
- Need specific PostgreSQL extensions (PostGIS, pgvector, TimescaleDB)
- Open-source preference / vendor lock-in concerns
- JSON-heavy workloads (PostgreSQL's JSONB is excellent for semi-structured alongside relational)
- Cost — PostgreSQL Flexible Server is often cheaper than equivalent SQL Database tiers
- Need built-in connection pooling (pgBouncer) for microservices with many connections
- AI/vector search workloads — pgvector integrates vector search into existing relational queries

### Read Replicas
- Asynchronous replication to up to 5 replicas (same or different region)
- Cross-region replicas for geo-read distribution and DR
- Replica promotes to standalone server on failover
- **Gotcha**: async replication means replicas may lag — not suitable for read-your-writes requirements

## Azure Database for MySQL Flexible Server

### Key Features
- MySQL engine compatibility (versions 5.7, 8.0+)
- Zone-redundant HA and same-zone HA (same model as PostgreSQL Flexible Server)
- Stop/start for dev/test cost savings
- Custom maintenance windows
- Read replicas for scale-out reads
- Data-in replication for hybrid scenarios

### When to Pick MySQL
- Application is MySQL-native: WordPress, Magento, Drupal, custom LAMP/LEMP stacks
- Team expertise is MySQL-specific
- Migrating from Amazon RDS for MySQL, Google Cloud SQL for MySQL, Percona MySQL, or Amazon Aurora MySQL
- All of these source types are supported for both offline and online migration via Azure DMS (GA status per MS Learn)

### MySQL vs PostgreSQL Decision
| Factor | PostgreSQL | MySQL |
|---|---|---|
| Extension ecosystem | Richer (PostGIS, pgvector, TimescaleDB) | More limited |
| JSON support | JSONB (binary, indexable, excellent) | JSON (text-based, improving) |
| Connection pooling | Built-in pgBouncer | External (ProxySQL) |
| Replication | Logical + physical | Binary log + GTID |
| Best for | Complex queries, GIS, AI/vector, analytics | Web apps, CMS, LAMP stacks |
| Community | Enterprise/academic/analytics strong | Web development strong |
