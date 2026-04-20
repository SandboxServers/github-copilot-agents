# Best Practices

Proven patterns for Azure database design, operations, and security.

## Service Selection Decision Matrix

| Factor | Azure SQL Database | Cosmos DB | PostgreSQL Flexible |
|---|---|---|---|
| Data model | Relational, strong schema | Document/key-value, flexible schema | Relational, extensible (JSONB, PostGIS) |
| Scale pattern | Vertical (vCore) + read replicas | Horizontal (partition key) + multi-region | Vertical + read replicas |
| Consistency | ACID transactions | 5 tunable levels (Strong→Eventual) | ACID transactions |
| Global distribution | Geo-replication (async) | Native multi-region writes | Geo-replication (async, read replicas) |
| Best for | LOB apps, ERP, structured data | IoT, gaming, catalogs, high-scale NoSQL | Open-source ecosystem, spatial, analytics |

## Connection Resilience

### Retry Policies
- All Azure PaaS databases produce transient faults — retry is mandatory, not optional
- Azure SQL: use `SqlRetryLogicBaseProvider` with exponential backoff (initial 1s, max 30s, 5 attempts)
- Cosmos DB: SDK has built-in retry — configure `MaxRetryAttemptsOnRateLimitedRequests` (default 9) and `MaxRetryWaitTimeInSeconds` (default 30)
- PostgreSQL: use Npgsql's `EnableRetryOnFailure()` or Polly for custom policies
- Always use idempotent operations so retries don't cause duplicate side effects
- Log transient fault retries — a spike in retries is an early indicator of capacity issues

### Connection Pooling
- ADO.NET pools by default — set `Max Pool Size=100` (or higher for high-throughput apps)
- Cosmos DB: use singleton `CosmosClient` — never create per-request instances
- PostgreSQL: use Npgsql connection pooling or PgBouncer for high-connection scenarios

### Circuit Breakers
- Wrap database calls with Polly circuit breaker to fail fast during prolonged outages
- Open circuit after 5 consecutive failures, half-open after 30 seconds
- Prevents connection pool exhaustion and cascading failures

## Security

### Authentication
- Use managed identity (system-assigned or user-assigned) — no connection string secrets
- Azure SQL: `Authentication=Active Directory Default` in connection string
- Cosmos DB: `DefaultAzureCredential` with RBAC data plane roles (`Cosmos DB Built-in Data Contributor`)
- PostgreSQL: use Entra ID authentication with `azure-identity` library
- Disable SQL authentication / access keys in production
- Rotate any remaining keys on a 90-day schedule with Azure Key Vault

### Network
- Deploy private endpoints for all databases — disable public network access
- Use NSG rules to restrict traffic to known application subnets
- For Cosmos DB, configure VNet service endpoints as a secondary control

### Encryption
- TDE (Transparent Data Encryption) is enabled by default on Azure SQL — verify it's not disabled
- Cosmos DB encrypts all data at rest by default with Microsoft-managed keys
- Use Always Encrypted for sensitive columns (SSN, credit card) — keys never exposed to the server
- Enable TLS 1.2 minimum for all connections

## Performance

### Indexing
- Azure SQL: review missing index DMVs weekly (`sys.dm_db_missing_index_details`)
- Rebuild indexes with >30% fragmentation, reorganize at 10-30%
- Cosmos DB: customize indexing policy — exclude paths not used in queries to reduce write RU cost
- PostgreSQL: use `pg_stat_user_indexes` to find unused indexes consuming write overhead
- Avoid over-indexing — each index adds write latency and storage cost
- Use columnstore indexes on Azure SQL for analytical queries over large fact tables

### Query Optimization
- Enable Query Store on Azure SQL — tracks query plans, regressions, and resource consumption
- Use Intelligent Insights for automated performance diagnostics and recommendations
- Cosmos DB: use `FeedOptions.PopulateQueryMetrics` to profile RU cost per query
- Avoid `SELECT *` — project only needed columns to reduce IO and network cost

## Cost Optimization
- Right-size: use `sys.dm_db_resource_stats` (SQL) or RU consumption metrics (Cosmos DB) to find actual usage
- Reserved capacity: commit 1-year or 3-year for stable workloads — saves 30-65% vs. pay-as-you-go
- Azure Hybrid Benefit: apply existing SQL Server licenses to vCore model — saves ~40%
- Cosmos DB: autoscale for variable workloads, serverless for dev/test, reserved capacity for steady production
- Elastic pools for multi-tenant with variable per-tenant usage
- Delete unused replicas, test databases, and expired snapshots
- Use Azure Advisor cost recommendations — flags over-provisioned and underutilized databases
- PostgreSQL: use burstable tiers (B-series) for dev/test, General Purpose for production
- Schedule non-production databases to auto-pause outside business hours

## Backup Strategy
- Azure SQL PITR: 7-35 day retention (default 7) — set to at least 14 for production
- Enable Long-Term Retention (LTR) for compliance: weekly, monthly, yearly backups up to 10 years
- Cosmos DB: enable continuous backup (30-day retention) — periodic backup (4hr/8hr) is legacy
- Test restores quarterly — restore to a different server/account, verify data integrity
- Geo-restore: verify restore works to a paired region — test during DR drills

## Monitoring and Alerting
- Alert on DTU/CPU >80% sustained for 15 minutes (Azure SQL)
- Alert on normalized RU consumption >70% (Cosmos DB) — indicates approaching throttling
- Monitor deadlocks (`sys.dm_exec_requests`), blocked processes, and failed connections
- Set up Azure Monitor diagnostic settings — route to Log Analytics for KQL queries
- Track connection count trends — sudden spikes indicate connection pool leaks
- Enable Intelligent Insights for automated root-cause analysis on Azure SQL performance regressions
- Cosmos DB: monitor `TotalRequestUnits` by operation type to find expensive queries
- Set up action groups for critical alerts — email + PagerDuty/ServiceNow for production
- Review Azure Advisor recommendations monthly — flags right-sizing, security, and HA gaps
