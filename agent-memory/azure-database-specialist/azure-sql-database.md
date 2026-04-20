# Azure SQL Database

## Purchasing Models

### DTU (Database Transaction Units)
- Bundled measure of CPU, memory, reads, writes in preconfigured packages
- Tiers: Basic (5 DTU), Standard (10-3000 DTU), Premium (125-4000 DTU)
- Storage included in price with extra storage available in Standard/Premium
- Best for predictable steady workloads wanting simplicity
- Cannot independently scale CPU vs memory vs storage — all-or-nothing bundling
- No serverless, no Azure Hybrid Benefit, no reserved instances, no Hyperscale

### vCore (Virtual Core) — Recommended
- Independent control of compute, memory, and storage
- Service tiers: General Purpose, Business Critical, Hyperscale
- Compute tiers: Provisioned (always-on) or Serverless (auto-scale, auto-pause)
- Azure Hybrid Benefit: bring existing SQL Server licenses for ~55% savings
- Reserved instances: 1-year (~33% savings) or 3-year (up to ~72% savings)
- Best for workloads needing fine-grained control or migration from on-prem SQL Server

### DTU vs vCore Quick Reference

| Aspect | DTU | vCore |
|---|---|---|
| Resource control | Bundled | Independent CPU/memory/storage |
| Cost transparency | Opaque | Clear (X cores, Y GB memory) |
| Serverless | No | Yes |
| Azure Hybrid Benefit | No | Yes |
| Reserved instances | No | Yes |
| Hyperscale | No | Yes |

Recommendation: vCore for new workloads. DTU only for small, predictable workloads wanting simplicity.

## Service Tiers (vCore)

### General Purpose
- Remote storage (Azure Premium Storage) — higher IO latency than Business Critical
- 99.99% SLA with zone redundancy
- Serverless option: auto-pause after configurable idle (min 1 hour), billed per-second
- Best for most workloads, dev/test, applications tolerant of slightly higher IO latency

### Business Critical
- Local SSD storage with low-latency IO and built-in read replica
- 3 Always On synchronous replicas — one usable as free read-only endpoint
- ~2.7x price of General Purpose (paying for 3 replicas)
- 99.995% SLA with zone redundancy
- In-memory OLTP available for extreme OLTP performance
- Best for latency-sensitive OLTP with high IO requirements

### Hyperscale
- Up to 100TB database size with distributed page server architecture
- Rapid scale-out of read replicas (up to 4 named replicas)
- Near-instant snapshot-based backups regardless of database size
- Fast restore in minutes not hours
- Serverless available: auto-scales up to 80 vCores
- Zone-redundant HA supported
- Best for large databases, variable workloads, databases outgrowing 4TB GP limit

### Tier Comparison

| Feature | General Purpose | Business Critical | Hyperscale |
|---|---|---|---|
| Max size | 4TB | 4TB | 100TB |
| Storage type | Remote | Local SSD | Multi-tiered |
| IO latency | Higher (remote) | Lowest (local) | 1-2ms local, 1-4ms remote |
| Built-in read replica | No | Yes (1 free) | Yes (0-4 named) |
| Serverless | Yes | No | Yes |
| In-memory OLTP | No | Yes | No |
| Backup speed | Size-dependent | Size-dependent | Near-instant |

## Elastic Pools
- Share resources (eDTUs or vCores) across multiple databases on same logical server
- Ideal when databases have different peak times with low average but high peak utilization
- Sizing formula (DTU): MAX(total_DBs × avg_DTU, concurrent_peak_DBs × peak_DTU)
- DTU model: pool eDTU price ~1.5x single DB but shared across many DBs = net savings
- vCore model: same unit price as single databases — savings from sharing
- Per-database min/max settings prevent one database consuming all pool resources
- SaaS pattern: one pool per tier, databases assigned by workload intensity
- Gotcha: if all databases peak simultaneously, pool does not save money
- Hyperscale databases cannot be placed in elastic pools

## Serverless Compute
- Auto-scales compute within configurable min/max vCore range
- Auto-pauses after idle period (min 1 hour) — only storage billed during pause
- Auto-resumes on first connection with cold start typically under 1 minute
- Billed per-second for compute actually used
- Available in General Purpose and Hyperscale tiers
- Cost math: if utilization above 60-70%, provisioned is cheaper; below that, serverless wins

## Key Features
- Automatic tuning: auto-create indexes, auto-drop unused indexes, force last known good plan
- Intelligent Insights: ML-powered detection of degrading queries and plan regressions
- Query Performance Insight: identify top resource-consuming queries
- Readable secondaries (Business Critical/Hyperscale) for read scale-out
- Geo-replication: up to 4 readable secondaries in any region
- Failover groups: automatic DNS failover with grace period for grouped databases
