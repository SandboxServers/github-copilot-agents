# Database Performance

## SQL Database Performance

### Query Performance Insight
- Tracks query plans, execution stats, and wait stats over time via Query Store
- Identify regressions by comparing current plan performance to historical baselines
- Force good plans: pin a known-good execution plan for a query
- First thing to check when performance degrades — was there a plan change?

### Automatic Tuning
- Auto-create indexes: detects missing indexes from workload patterns, creates and validates
- Auto-drop unused indexes: removes indexes not used (with safety rollback period)
- Auto-force last known good plan: when plan regression detected, forces previous good plan
- All actions can be reviewed and reverted in Azure portal

### Indexing Strategy
- Missing index recommendations from DMVs, Query Store, and Intelligent Insights
- Columnstore indexes for analytical queries on OLTP databases (hybrid transactional/analytical)
- Too many indexes slow down writes — every insert/update/delete maintains all indexes

### Wait Stats Analysis
- Identify bottleneck category: CPU, IO, memory, locking, network
- Key waits to watch:
  - RESOURCE_SEMAPHORE — memory pressure
  - PAGEIOLATCH — IO bottleneck (remote storage latency in General Purpose)
  - LCK — locking and blocking
  - LOG_RATE_GOVERNOR — hitting log rate limit (common in General Purpose)

### Intelligent Insights
- ML-powered detection of performance issues
- Detects: degrading queries, plan regressions, memory pressure, blocking, tempdb contention
- Produces human-readable explanations with root cause analysis

### In-Memory OLTP
- Available on Business Critical tier only
- Memory-optimized tables and natively compiled stored procedures
- Eliminates latch contention for extreme OLTP throughput

## Cosmos DB Performance

### Partition Key — Number One Performance Factor
- Bad partition key = hot partitions = 429 throttling even with high RU/s
- Monitor per-partition RU consumption — uneven distribution indicates problems
- Cross-partition queries are expensive: fan out to all partitions, aggregate results
- Point reads (ID + partition key) = 1 RU per 1 KB — always prefer this pattern

### Indexing Policy
- Default: all properties indexed (good for reads, expensive for writes)
- Exclude write-heavy paths never queried — reduces RU cost on writes
- Add composite indexes for ORDER BY on multiple properties
- Spatial indexes for geospatial queries (point, polygon, linestring)

### Integrated Cache
- In-memory cache for hot data within Cosmos DB dedicated gateway
- Reduces RU consumption for frequently read items
- Best for read-heavy workloads with hot partition patterns

### SDK Best Practices
- Use Direct mode connectivity (not Gateway) for lowest latency
- Single CosmosClient instance per application lifetime
- Enable TCP connection multiplexing
- Use FeedIterator for large result sets — stream instead of loading all into memory

### Consistency Level Impact
- Session = 1x RU for reads (sweet spot for most apps)
- Strong/Bounded Staleness = 2x RU for reads
- Downgrading from Strong to Session = 50% read cost reduction

## PostgreSQL Performance

### Connection Pooling (PgBouncer)
- Built-in PgBouncer on Flexible Server — critical for high-connection microservices
- Prevents connection exhaustion (PostgreSQL connections are heavyweight)
- Transaction pooling mode recommended for most workloads

### Query Optimization
- EXPLAIN ANALYZE to understand query plans and actual execution times
- Identify sequential scans on large tables — add appropriate indexes
- pg_stat_statements extension for tracking query performance over time
- Partial indexes for frequently filtered subsets
- GIN indexes on JSONB columns for semi-structured data queries

### Vacuum Tuning
- Autovacuum settings critical for maintaining query performance
- Monitor dead tuple counts and autovacuum lag
- Tune autovacuum_vacuum_threshold and autovacuum_vacuum_scale_factor

## General Best Practices
- Connection pooling always — all database types benefit from connection reuse
- Retry logic with exponential backoff for transient failures
- Read replicas for read-heavy workloads to offload primary
- Caching layer (Redis) for hot data to reduce database load
- Query parameterization to avoid plan cache pollution
- Monitor and alert on key metrics: CPU, IO, memory, connection count, replication lag
