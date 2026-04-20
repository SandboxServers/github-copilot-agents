# Gotchas

Surprising behaviors and hidden limits in Azure database services. These catch experienced engineers.

## Azure SQL: DTU Limits Are Bundled
- DTU caps CPU, IO, and memory simultaneously — hitting any one limit throttles the whole database
- A CPU-bound query can be throttled even if IO and memory are idle
- `sys.dm_db_resource_stats` shows percentage of DTU consumed but doesn't break down which resource is the bottleneck
- **Trap:** S3 (100 DTU) has only 200 concurrent requests — not documented prominently
- **Trap:** tempdb performance is also limited by DTU tier — heavy temp table usage hits DTU ceiling faster
- Monitor `avg_cpu_percent`, `avg_data_io_percent`, `avg_log_write_percent` to find the real limiter
- If one metric consistently maxes out while others are low, vCore model gives independent resource control

## Cosmos DB: 404 Responses Still Cost RUs
- A point read returning "Not Found" costs ~1 RU — same as a successful 1KB read
- High-volume existence checks (polling for a document that doesn't exist yet) burn RUs silently
- Queries returning 0 results still consume RUs for the index scan
- Replace operations on non-existent documents cost RUs and return 404
- **Mitigation:** use change feed instead of polling, cache known-missing keys client-side
- **Mitigation:** use conditional operations (`If-Match` headers) to avoid blind replace attempts

## Azure SQL Serverless: Auto-Pause Cold Start
- After inactivity (configurable 1hr–7 days), the database pauses and stops billing compute
- First connection after pause takes 30-60 seconds to resume — user-visible latency spike
- The connection attempt may timeout if client timeout < 60 seconds
- **Trap:** health check probes keep the database awake, defeating the cost savings
- **Trap:** Azure Functions consumption plan + SQL serverless = double cold start stacking
- Always On availability groups and geo-replication are not supported with auto-pause
- Set `Connection Timeout=120` in connection strings for serverless databases
- Consider using minimum vCores > 0 to avoid full pause while still getting autoscale savings

## PostgreSQL Flexible: Major Version Upgrades
- In-place major version upgrade (e.g., 14→16) requires a maintenance window with downtime
- Extensions must be compatible with the target version — check `pg_available_extensions` first
- Custom `shared_preload_libraries` settings may be reset during upgrade
- Logical replication slots are dropped during major upgrades
- Custom parameter group settings should be documented — some parameters change defaults between major versions
- Large databases (>1TB) can take hours for the upgrade process to complete
- **Mitigation:** test upgrade on a restored copy first, verify all extensions, plan a maintenance window
- **Mitigation:** use read replica promotion for zero-downtime major version upgrades (create replica on new version, promote, switch DNS)

## Cosmos DB: Consistency Levels Affect Cost
- Strong/Bounded Staleness reads cost 2× RUs (two-replica quorum reads)
- Session consistency is the default — good balance for most applications
- Changing consistency level at the account level affects all containers globally
- Per-request consistency downgrade is possible (e.g., account=Strong, request=Eventual) but upgrade is not
- Strong consistency with multi-region writes is not supported — must use single-write region
- Bounded staleness in multi-region has higher write latency due to cross-region replication acknowledgment
- **Tip:** for most web/API workloads, Session consistency provides read-your-writes without the 2× RU penalty

## Redis Cache: Clustering Breaks Multi-Key Operations
- Clustered Redis distributes keys across shards by hash slot
- `MGET`, `MSET`, `SUNION`, and Lua scripts fail if keys span different shards
- Transactions (`MULTI/EXEC`) only work within a single shard
- **Trap:** an app working perfectly on non-clustered Premium tier breaks after enabling clustering
- **Trap:** key migration during cluster scaling can cause brief key unavailability
- Pipelines are shard-aware in StackExchange.Redis but not all client libraries handle this transparently
- **Fix:** use hash tags `{user:123}.profile` and `{user:123}.prefs` to co-locate related keys
- **Fix:** design key naming conventions upfront that support future clustering needs

## Azure SQL: Elastic Pool eDTU Sharing
- eDTUs are a shared pool — one hot database can consume most of the pool's resources
- Per-database max eDTU limits cap individual databases but don't guarantee minimum
- A noisy-neighbor database running a large report can starve OLTP databases in the same pool
- **Mitigation:** set per-database min/max eDTU, monitor with `sys.elastic_pool_resource_stats`, isolate heavy databases

## Cosmos DB: TTL Deletes Consume RUs
- Expired documents are deleted in a background process that consumes provisioned RUs
- Bulk expiration (millions of documents expiring simultaneously) can cause 429 throttling for normal operations
- TTL deletions don't appear in change feed (by default) — downstream systems miss delete events
- TTL is set at the container level but can be overridden per document with `_ttl` property
- **Mitigation:** stagger TTL values, provision headroom for bulk expiry, use "all versions and deletes" change feed mode

## Azure SQL: Geo-Replication Is Async
- Active geo-replication uses asynchronous commit — secondary may lag seconds behind primary
- RPO is typically <5 seconds but not guaranteed under heavy load
- Failover to secondary may lose the last few committed transactions
- Auto-failover groups provide DNS endpoint redirection but add ~30 seconds failover time
- **Trap:** applications reading from geo-secondary see stale data — not suitable for consistency-critical reads
- **Trap:** after failover, the old primary must be re-seeded — can take hours for large databases
- Monitor replication lag with `sys.dm_geo_replication_link_status` — alert if lag exceeds your RPO target

## Managed Instance: Subnet Requirements
- Requires a dedicated subnet with minimum /27 (32 addresses), recommended /26
- Subnet must be delegated to `Microsoft.Sql/managedInstances` — no other resources allowed
- NSG rules are auto-managed — custom rules can block management plane traffic and cause failures
- Changing subnet after deployment requires migration (create new MI, restore, switch DNS)
- UDR (User-Defined Routes) must include management plane routes — missing routes cause deployment failures
- Adding more instances to a subnet requires spare IPs — plan for growth beyond the initial instance
- **Mitigation:** plan subnet CIDR generously upfront, use /24 for growth, document NSG and UDR dependencies
- **Mitigation:** use the `Microsoft.Sql/managedInstances/subnetCheck` API to validate subnet before deployment
