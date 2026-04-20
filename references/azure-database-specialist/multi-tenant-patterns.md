# Multi-Tenant Database Patterns

Patterns and tradeoffs for multi-tenant database design on Azure.

## Isolation Models

| Model | Isolation | Cost | Complexity | Best For |
|---|---|---|---|---|
| Database-per-tenant | Full | High (one DB per tenant) | Medium | Enterprise SaaS, strict compliance |
| Schema-per-tenant | Schema-level | Medium | High | Mid-market SaaS, moderate isolation |
| Shared database, shared schema | Row-level (tenant ID column) | Low | Low | High-volume consumer apps |
| Elastic pool + DB-per-tenant | Full + shared resources | Medium | Medium | Variable workload SaaS |
| Cosmos DB container-per-tenant | Container-level | High | Medium | NoSQL multi-tenant with tenant-level scaling |
| Cosmos DB shared container | Partition-key-level | Low | Low | High tenant count, cost-sensitive |

## Azure SQL: Elastic Pool Pattern
- Group tenant databases in an elastic pool to share eDTU/vCore across tenants
- Tenants with staggered peak usage patterns benefit most from pooling
- Set per-database min/max eDTU to prevent noisy-neighbor starvation
- Shard map manager (Elastic Database tools) routes connections to the correct tenant database
- Use Elastic Jobs to run maintenance (index rebuilds, stats updates) across all tenant databases
- **Scaling:** add pools as tenant count grows, rebalance databases across pools
- **Monitoring:** track per-database DTU consumption — databases consistently hitting max should be isolated

## Azure SQL: Row-Level Security (RLS)
- Single shared database with a `TenantId` column on every table
- SQL Server Row-Level Security filters rows by `SESSION_CONTEXT('TenantId')`
- Application sets `sp_set_session_context @key='TenantId', @value=@tenantId` on each connection
- **Risk:** if the app fails to set the context, a tenant sees all data — must enforce at middleware level
- Combine with indexed `TenantId` columns for performance (avoid full table scans)

## Cosmos DB: Hierarchical Partition Keys
- Use hierarchical partition keys for multi-tenant: `/tenantId` → `/category` → `/id`
- Avoids the 20GB logical partition limit for large tenants
- Single container serves all tenants — simplifies operations and cost management
- Query with tenant-scoped partition key to avoid cross-partition fan-out
- Per-tenant throughput isolation is not possible in a shared container — all tenants share RU pool
- **Cost:** RU consumption is per container — use autoscale to handle variable per-tenant load
- **Alternative:** for strict tenant isolation, use one container per tenant with dedicated throughput

## Tenant Provisioning and Lifecycle
- Automate tenant onboarding: ARM/Bicep templates or Terraform modules to provision tenant databases
- Store tenant-to-database mapping in a catalog database or Azure App Configuration
- Implement tenant deactivation (soft delete) before hard deletion — retain data per SLA
- Schema migrations: use EF Core migrations or Flyway with a deployment pipeline that targets all tenant databases
- Provision new tenants in the pool/region with the most headroom — use catalog metadata for placement decisions
- Implement tenant-aware connection string resolution in the application middleware layer
- Support tenant export (data portability) for compliance with GDPR right-to-data-portability

## Data Residency and Compliance
- Some tenants require data to stay in a specific region (GDPR, data sovereignty)
- Database-per-tenant model supports placing individual databases in specific regions
- Shared database model requires row-level region tagging and routing logic
- Cosmos DB: per-container geo-replication configuration can differ — but all data in a container goes to the same regions
- Use Azure Policy to enforce allowed regions for database resource creation
- Document data residency requirements per tenant in the catalog database for audit trails

## Monitoring Multi-Tenant Databases
- Track per-tenant resource consumption: DTU/RU per tenant, query count, storage size
- In shared-schema models, use query tags or session context to attribute cost to tenants
- Alert on tenant-level anomalies: sudden spike in RU usage, storage growth, or error rates
- Elastic pool: monitor pool-level utilization and per-database hot spots with `sys.elastic_pool_resource_stats`
- Build tenant usage dashboards for billing (chargeback) and capacity planning
- Log tenant ID on every database operation for debugging and audit purposes

## Anti-Patterns (Multi-Tenant Specific)
- Table-per-tenant in a single database: unmanageable at scale, schema migrations become impossible
- Column-level tenant customization: adding columns for specific tenants pollutes the shared schema
- No tenant isolation testing: production data leaks between tenants are the highest-severity SaaS bugs
- Manual schema changes: always use automated pipelines to deploy schema changes across all tenant databases
- Shared admin credentials across tenants: use per-tenant managed identities or scoped RBAC roles
- No tenant-level backup/restore: ensure you can restore a single tenant's data without affecting others
