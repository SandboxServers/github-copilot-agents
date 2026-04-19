# Performance Tuning

## SQL Database Performance Tuning

### Query Store
- Tracks query plans, execution stats, and wait stats over time
- Identify regressions: compare current plan performance to historical baselines
- Force good plans: pin a known-good execution plan for a query
- Available in all service tiers
- **First thing to check** when performance degrades — was there a plan change?

### Indexing
- Missing index recommendations from DMVs, Query Store, and Intelligent Insights
- **Automatic tuning**: auto-create indexes based on workload patterns, auto-drop unused indexes
- Columnstore indexes for analytical queries on OLTP databases (hybrid transactional/analytical)
- **Gotcha**: too many indexes slow down writes. Every insert/update/delete must maintain all indexes.

### Wait Stats
- Identify bottleneck category: CPU, IO, memory, locking, network
- Key waits to watch:
  - `RESOURCE_SEMAPHORE` — memory pressure
  - `PAGEIOLATCH_*` — IO bottleneck (remote storage latency in General Purpose)
  - `LCK_*` — locking/blocking
  - `CXPACKET/CXCONSUMER` — parallelism overhead
  - `LOG_RATE_GOVERNOR` — hitting log rate limit (common in General Purpose)

### Intelligent Insights
- ML-powered detection of performance issues
- Detects: degrading queries, plan regressions, memory pressure, blocking, tempdb contention
- Available via Azure Monitor Diagnostics
- Produces human-readable explanations with root cause analysis

### Automatic Tuning
- **Auto-create indexes**: detects missing indexes from workload patterns, creates and validates
- **Auto-drop unused indexes**: removes indexes that haven't been used (with safety rollback period)
- **Auto-force last known good plan**: when plan regression detected, forces previous good plan
- All automatic actions can be reviewed and reverted in Azure portal

### Database Advisor
- Recommendations for: indexing, parameterization issues, schema issues
- Impact estimates for each recommendation
- Can be configured to auto-apply recommendations

### Hyperscale-Specific Performance
- Local SSD IOPS scale with vCores: from 8,000 (2 vCores) to 204,800 (80 vCores serverless)
- Local read IO latency: 1-2ms, Remote read IO latency: 1-4ms, Write IO latency: 1-4ms
- Max log rate: 100 MiB/s across all serverless sizes
- Max concurrent workers scale with vCores (150 at 2 vCores to 6,000 at 80 vCores)

## Cosmos DB Performance

### Partition Key (Single Most Important Decision)
- Bad partition key = hot partitions = throttling (429 responses) even with high RU/s
- Monitor per-partition RU consumption — uneven distribution indicates partition key problems
- Cross-partition queries are expensive: fan out to all partitions, aggregate results
- Point reads (ID + partition key) = 1 RU per 1 KB — always prefer this pattern

### Indexing Policy
- Default: all properties indexed (good for reads, expensive for writes)
- Exclude write-heavy paths you never query — reduces RU cost on writes
- Add composite indexes for `ORDER BY` on multiple properties — without them, query fails or is very expensive
- Spatial indexes for geospatial queries (point, polygon, linestring)

### SDK Best Practices
- Use **Direct mode** connectivity (not Gateway mode) for lowest latency
- Keep SDK updated — performance improvements in each release
- Use connection pooling and a **single CosmosClient instance** per application lifetime
- Enable TCP connection multiplexing
- Use `FeedIterator` for large result sets — stream results instead of loading all into memory

### Consistency Level Impact
- Session = 1x RU for reads (sweet spot for most apps)
- Strong/Bounded Staleness = 2x RU for reads (minority quorum requires reading from 2 replicas)
- Downgrading from Strong to Session for appropriate workloads = 50% read cost reduction

## PostgreSQL Performance

### Connection Pooling (pgBouncer)
- Built-in pgBouncer on Flexible Server — critical for high-connection microservices
- Prevents connection exhaustion (PostgreSQL connections are heavyweight)
- Transaction pooling mode recommended for most workloads

### Query Optimization
- `EXPLAIN ANALYZE` to understand query plans
- Identify sequential scans on large tables — add appropriate indexes
- Partial indexes for frequently filtered subsets
- JSONB indexing (GIN indexes) for semi-structured data queries
- `pg_stat_statements` extension for tracking query performance over time
