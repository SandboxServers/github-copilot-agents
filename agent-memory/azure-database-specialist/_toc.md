# Azure Database Specialist — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Data Store Selection | [data-store-selection.md](data-store-selection.md) | When choosing between database services, comparing options, or recommending a data platform |
| Azure SQL Database | [azure-sql-database.md](azure-sql-database.md) | When working with SQL Database — purchasing models, tiers, elastic pools, serverless |
| PostgreSQL & MySQL | [postgresql-mysql.md](postgresql-mysql.md) | When working with PostgreSQL Flexible Server or MySQL Flexible Server |
| Cosmos DB | [cosmos-db.md](cosmos-db.md) | When working with Cosmos DB — RU math, partition keys, consistency levels, APIs |
| Azure Cache for Redis | [redis-cache.md](redis-cache.md) | When working with caching, session state, pub/sub, or Redis architecture |
| Replication & Disaster Recovery | [replication-dr.md](replication-dr.md) | When designing HA, DR, geo-replication, failover groups, or RPO/RTO strategies |
| Backup & Restore | [backup-restore.md](backup-restore.md) | When designing backup strategies, PITR, LTR, geo-restore, or compliance retention |
| Security | [security.md](security.md) | When designing database security — encryption, authentication, masking, audit, threat detection |
| Performance Tuning | [performance.md](performance.md) | When diagnosing or optimizing database performance — indexing, query tuning, wait stats |
| Migration | [migration.md](migration.md) | When planning database migration — DMS, offline vs online, assessment, cutover |
| Cost Optimization | [cost-optimization.md](cost-optimization.md) | When optimizing database costs — right-sizing, reservations, hybrid benefit, Cosmos DB traps |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing designs for common database mistakes — bad partition keys, missing pooling, no retries |
| Gotchas | [gotchas.md](gotchas.md) | When debugging surprising behavior — DTU bundling, 404 RU cost, cold starts, TTL RU consumption |
| Best Practices | [best-practices.md](best-practices.md) | When designing end-to-end — service selection, connection resilience, security, performance, cost |
| Multi-Tenant Patterns | [multi-tenant-patterns.md](multi-tenant-patterns.md) | When designing multi-tenant SaaS — isolation models, elastic pools, RLS, hierarchical partition keys |
