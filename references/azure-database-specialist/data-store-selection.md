# Data Store Selection

## Decision Matrix

| Service | Model | Best For | Max Size | SLA |
|---|---|---|---|---|
| SQL Database | Relational | OLTP, structured data, complex queries | 100TB (Hyperscale) | 99.995% (zone-redundant) |
| PostgreSQL Flexible | Relational | Open-source, PostGIS, JSON, extensions | 64TB | 99.99% (zone-redundant) |
| MySQL Flexible | Relational | WordPress, CMS, LAMP stack | 16TB | 99.99% (zone-redundant) |
| Cosmos DB | Multi-model | Global distribution, low latency, variable schema | Unlimited | 99.999% (multi-region) |
| Redis Cache | Key-value | Session state, caching, real-time leaderboards | 120GB (P5) | 99.9% |
| Table Storage | Key-value | Cheap key-value, logs, IoT telemetry | 500TB/table | Storage SLA |

## Decision Tree

```
Need global distribution with multi-region writes?        → Cosmos DB
Need relational with strong consistency and T-SQL?        → Azure SQL Database
Need relational with open-source and extensions?          → PostgreSQL Flexible Server
Need MySQL compatibility (WordPress, CMS, LAMP)?          → MySQL Flexible Server
Need sub-millisecond caching or session state?            → Azure Cache for Redis
Need cheap key-value storage at scale?                    → Table Storage
Need data warehouse or massive analytics?                 → Synapse Analytics / Fabric
Need vector similarity search?                            → Cosmos DB MongoDB vCore or PostgreSQL pgvector
```

## When to Choose Each

### Azure SQL Database
- Enterprise OLTP with complex relational schemas and stored procedures
- T-SQL expertise on the team with SSMS, Azure Data Studio, Entity Framework tooling
- Advanced security features: Always Encrypted, Ledger tables, Row-Level Security
- Hyperscale tier for databases up to 100TB with near-instant backups
- Serverless compute for intermittent workloads with auto-pause billing
- Elastic pools for SaaS multi-tenant patterns sharing resources

### Cosmos DB
- Global distribution is a hard requirement with multi-region read and write
- Variable or evolving schema with document model flexibility
- Single-digit millisecond latency at any scale with 99.999% SLA
- Multi-model needs: NoSQL, MongoDB, Cassandra, Gremlin, Table APIs
- Event-driven or IoT workloads with massive write throughput

### PostgreSQL Flexible Server
- Application is PostgreSQL-native or team has PostgreSQL expertise
- Need extensions: PostGIS (geospatial), pgvector (AI/vector search), TimescaleDB (time-series)
- JSON-heavy workloads with JSONB for semi-structured data
- Open-source preference or vendor lock-in concerns
- Built-in PgBouncer connection pooling for high-connection workloads
- Often cheaper than equivalent SQL Database tiers

### MySQL Flexible Server
- WordPress, Magento, Drupal, or custom LAMP/LEMP stacks
- Migrating from Amazon RDS MySQL, Google Cloud SQL MySQL, or on-prem
- Team expertise is MySQL-specific

### Azure Cache for Redis
- Caching layer in front of a primary database (session state, output caching)
- Real-time pub/sub messaging, distributed locking, leaderboards
- Redis modules (RediSearch, RedisJSON) require Enterprise tier

### Table Storage
- Simple key-value storage at very low cost
- IoT telemetry, logs, diagnostic data with no complex query needs

## Anti-Patterns
- Using Cosmos DB when a simple relational database suffices — cost explosion
- Using SQL Database for truly schemaless data — forcing relational on documents
- Using Redis as a primary database — it is a cache, not a system of record
- Choosing based on familiarity rather than workload fit
- Ignoring total cost of ownership — cheap per-unit times many units equals expensive

## Approximate Monthly Starting Costs

| Service | Dev/Test Entry | Production Starting Point |
|---|---|---|
| SQL Database | ~$5/mo (serverless, paused) | ~$150/mo (GP 2 vCores) |
| PostgreSQL Flexible | ~$13/mo (Burstable B1ms) | ~$100/mo (GP D2ds_v5) |
| MySQL Flexible | ~$13/mo (Burstable B1ms) | ~$100/mo (GP D2ds_v5) |
| Cosmos DB | Free tier (1000 RU/s) | ~$25/mo (serverless light) |
| Redis | ~$17/mo (Basic C0) | ~$40/mo (Standard C1) |
| Table Storage | ~$0.05/GB/mo | ~$0.05/GB/mo |
