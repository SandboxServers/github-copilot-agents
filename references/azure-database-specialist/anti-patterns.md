# Anti-Patterns

Common Azure database mistakes. If you see these in a design or PR, flag them.

## Wrong Cosmos DB Partition Key
- Choosing a low-cardinality key (e.g., `status`, `country`) creates hot partitions
- Hot partitions hit RU throttling (429s) while other partitions sit idle — provisioned throughput is per physical partition
- Cross-partition queries (queries without partition key filter) fan out to all partitions — RU cost and latency multiply
- Partition key cannot be changed after container creation — must recreate the container and migrate data
- Synthetic partition keys (concatenated fields) seem clever but complicate point reads
- **Fix:** choose a high-cardinality key that appears in most query predicates (e.g., `tenantId`, `userId`)
- **Fix:** use hierarchical partition keys for multi-dimensional access patterns

## Over-Provisioning Cosmos DB RUs
- Provisioning for peak load 24/7 when traffic is variable wastes 50-80% of spend
- Manual provisioned throughput doesn't scale down automatically
- Multi-region deployments multiply the over-provisioning cost (RU/s × region count)
- Teams set high RU/s during load testing and forget to scale back down
- Shared throughput databases spread RUs across containers — one hot container starves others
- **Fix:** use autoscale (scales 10-100% of max) for variable workloads, serverless for dev/test
- **Fix:** monitor normalized RU consumption metrics to right-size, use reserved capacity for stable baselines

## Not Using Connection Pooling
- Opening a new connection per request exhausts SQL Database connection limits (max varies by tier: Basic=30, S0=60, P1=200)
- Connection establishment overhead adds 20-50ms per request (TCP handshake + TLS + auth)
- Cosmos DB SDK manages connections internally, but misconfiguring `MaxConnectionsPerEndpoint` causes throttling
- PostgreSQL: default `max_connections` on Flexible Server varies by tier — exceeding it returns "too many connections"
- Connection pool exhaustion manifests as intermittent timeouts, not clear error messages
- **Fix:** use built-in pooling (ADO.NET pools by default), configure `Max Pool Size` in connection strings
- **Fix:** for microservices with many instances, use connection pool sizing formula: `max_pool = max_connections / app_instances`

## No Backup Verification
- Azure SQL PITR exists but teams never test restoring from it
- Cosmos DB continuous backup has a 30-day window — if corruption goes unnoticed longer, data is lost
- LTR (Long-Term Retention) policies configured but never validated with a restore drill
- **Fix:** schedule quarterly restore tests, document RPO/RTO, test geo-restore to a different region

## Storing Blobs in SQL
- Storing images, PDFs, or large binary data in `VARBINARY(MAX)` columns bloats database size
- Increases backup time, PITR storage cost, and replication lag
- DTU/vCore resources consumed for blob I/O instead of transactional queries
- Blob data in SQL inflates geo-replication bandwidth and increases failover time
- Common symptom: database size grows faster than row count — check for large binary columns
- **Fix:** store files in Azure Blob Storage, keep only the blob URL/reference in SQL
- **Fix:** for existing data, migrate blobs out with a background process, replace with Blob Storage URIs

## No Read Replicas for Read-Heavy Workloads
- Running all reads and writes against the primary replica causes unnecessary contention
- Azure SQL supports up to 4 read replicas (Business Critical / Hyperscale)
- Cosmos DB multi-region reads are built-in but must be enabled with `ApplicationPreferredRegions`
- PostgreSQL Flexible Server supports read replicas across regions — useful for geo-distributed reads
- Hyperscale named replicas allow independent scaling and connection strings per replica
- **Fix:** use `ApplicationIntent=ReadOnly` in connection strings for reporting/analytics queries
- **Fix:** for Cosmos DB, configure `ApplicationPreferredRegions` to prefer the nearest read region

## Ignoring Index Maintenance
- SQL index fragmentation above 30% degrades scan performance — Azure SQL doesn't auto-defragment
- Missing index DMV recommendations ignored, causing full table scans
- Cosmos DB: default "index everything" policy wastes RUs on write-heavy containers with few query patterns
- **Fix:** schedule `ALTER INDEX REBUILD` for SQL, customize Cosmos DB indexing policy to include only queried paths

## Using DTU When vCore Is Better
- DTU model bundles CPU, IO, memory into opaque units — no control over individual resource allocation
- At higher tiers (P4+), vCore pricing is often 20-40% cheaper with equivalent performance
- DTU doesn't support Azure Hybrid Benefit (existing SQL Server licenses)
- **Fix:** use `sys.dm_db_resource_stats` to profile actual resource usage, migrate to vCore if CPU or IO dominates

## No Threat Detection or Defender
- SQL Database without Microsoft Defender for SQL leaves injection attacks and anomalous access undetected
- Cosmos DB Defender detects anomalous access patterns, SQL injection variants, and suspicious key listing
- Threat detection is ~$15/server/month — trivial vs. breach cost
- **Fix:** enable Defender for SQL and Defender for Cosmos DB on all production accounts

## Single Database Instead of Elastic Pool
- Multi-tenant SaaS with one database per tenant, each provisioned individually, wastes DTUs during off-peak
- Tenants have staggered usage patterns — pooling smooths cost
- Elastic pool eDTUs are shared: 100 databases × 5 eDTU average ≈ 500 eDTU pool vs. 100 × 20 eDTU individual
- **Fix:** use elastic pools for databases with variable, non-overlapping usage patterns

## Missing Retry Logic
- Azure SQL transient faults (error codes 40613, 40197, 40501, 49918) are expected in PaaS
- Cosmos DB returns 429 (Request Rate Too Large) under throttling — must honor `x-ms-retry-after-ms`
- Applications without retry logic surface intermittent 500 errors to users
- PostgreSQL Flexible Server: transient errors during HA failover last 20-40 seconds
- Redis: brief connection drops during patching or scaling require reconnection logic
- **Fix:** use `Microsoft.Data.SqlClient` with `SqlRetryLogicOption`, Cosmos DB SDK has built-in retry — configure `MaxRetryAttemptsOnRateLimitedRequests`
- **Fix:** implement Polly retry + circuit breaker for a unified resilience strategy across all database calls

## No Private Endpoints
- Database accessible over public internet, even with firewall rules, expands the attack surface
- Service endpoints restrict to VNet but traffic still traverses the Azure backbone publicly
- Private endpoints assign a private IP from your VNet — traffic never leaves the private network
- Each database service needs its own private endpoint (SQL, Cosmos DB, Redis, PostgreSQL)
- Private DNS zones must be configured correctly — stale DNS entries cause connection failures after failover
- **Fix:** deploy private endpoints for all production databases, disable public network access
- **Fix:** use Azure Policy to deny creation of databases without private endpoints
