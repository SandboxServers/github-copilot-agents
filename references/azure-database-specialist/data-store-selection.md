# Data Store Selection

## Decision Tree

```
Relational data, ACID transactions, complex queries?       → Azure SQL Database / SQL Managed Instance
PostgreSQL compatibility, extensions, open-source?          → Azure Database for PostgreSQL Flexible Server
MySQL compatibility, WordPress/LAMP stacks?                → Azure Database for MySQL Flexible Server
Global distribution, variable schema, multi-model?          → Cosmos DB
Caching, session state, pub/sub, leaderboards?              → Azure Cache for Redis / Azure Managed Redis
Cheap key-value at scale?                                   → Table Storage / Cosmos DB Table API
Unstructured files, images, backups?                        → Blob Storage
Data warehouse, massive analytical queries?                 → Synapse Analytics / Fabric
Vector similarity search with NoSQL?                        → Cosmos DB for MongoDB vCore (DiskANN / HNSW indexes)
Vector similarity search with relational?                   → PostgreSQL Flexible Server (pgvector extension)
```

## Selection Criteria Deep Dive

### When to Choose Azure SQL Database
- Enterprise applications with complex relational schemas, stored procedures, and T-SQL expertise
- Need for advanced security features: Always Encrypted with secure enclaves, Ledger tables, Row-Level Security
- Hyperscale tier needed for databases up to 128 TB with near-instant backups
- Serverless compute for intermittent workloads with auto-pause (only storage billed during pause)
- Elastic pools for SaaS multi-tenant patterns where databases share resources
- Strong ecosystem: SSMS, Azure Data Studio, SSDT, Entity Framework, Dapper

### When to Choose PostgreSQL Flexible Server
- Application is PostgreSQL-native or team has PostgreSQL expertise
- Need for specific extensions: PostGIS (geospatial), pgvector (vector search/AI), TimescaleDB (time-series), pg_trgm (fuzzy text search)
- JSON-heavy workloads — PostgreSQL JSONB is excellent for semi-structured data alongside relational
- Open-source preference or vendor lock-in concerns
- Built-in pgBouncer connection pooling for high-connection workloads
- Cost — often cheaper than equivalent Azure SQL Database tiers
- Vector Search + Azure AI Extension for AI-powered applications

### When to Choose MySQL Flexible Server
- Application is MySQL-native: WordPress, Magento, Drupal, custom LAMP/LEMP stacks
- Migrating from Amazon RDS for MySQL, Google Cloud SQL for MySQL, or on-prem MySQL
- Team expertise is MySQL-specific
- Zone-redundant HA and same-zone HA available (same model as PostgreSQL Flexible Server)

### When to Choose Cosmos DB
- Global distribution is a hard requirement (multi-region read/write)
- Variable or evolving schema — document model flexibility
- Single-digit millisecond latency at any scale with 99.999% SLA (multi-region)
- Multi-model needs: document (NoSQL), MongoDB, Cassandra, Gremlin (graph), Table, PostgreSQL APIs
- Event-driven or IoT workloads with massive write throughput
- Hierarchical partition keys for fine-grained data distribution (up to 3 levels of subpartitioning)

### When to Choose Redis
- Caching layer in front of a primary database (session state, output caching, data cache)
- Real-time pub/sub messaging, distributed locking, leaderboards/counters
- Azure Managed Redis (new): based on Redis Enterprise, all tiers clustered by default, three performance tiers (Memory Optimized 8:1, Balanced 4:1, Compute Optimized 2:1) plus Flash Optimized for large datasets on NVMe
- Azure Cache for Redis (classic): Basic/Standard/Premium/Enterprise/Enterprise Flash tiers
- Need for Redis modules (RediSearch, RedisJSON, RedisTimeSeries, RedisBloom) — requires Enterprise tier or Azure Managed Redis

### Anti-Patterns to Watch For
- Using Cosmos DB when a simple relational database would suffice — over-engineering and cost explosion
- Using SQL Database for truly schemaless data — forcing a relational model on document data
- Using Redis as a primary database — it's a cache/data structure store, not a system of record
- Choosing a database based on familiarity rather than workload fit
- Ignoring total cost of ownership — a cheaper per-unit price with 10x the units is more expensive
