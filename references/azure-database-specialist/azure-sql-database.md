# Azure SQL Database

## Purchasing Models

### DTU (Database Transaction Units)
- Bundled measure of CPU, memory, reads, writes — simple, preconfigured packages
- Tiers: **Basic**, **Standard** (S0-S12), **Premium** (P1-P15)
- Storage included in price; extra storage available in Standard/Premium
- **Best for**: workloads with predictable, steady resource needs; customers wanting simplicity
- **Gotcha**: can't independently scale CPU vs memory vs storage — it's all-or-nothing bundling
- **Gotcha**: no serverless, no Azure Hybrid Benefit, no reserved instances, no Hyperscale

### vCore (Virtual Core) — Recommended
- Independent control of compute, memory, and storage
- Service tiers: **General Purpose**, **Business Critical**, **Hyperscale**
- Compute tiers: **Provisioned** (always-on) or **Serverless** (auto-scale, auto-pause)
- Hardware: Standard-series (Gen5) for both provisioned and serverless
- **Azure Hybrid Benefit**: bring existing SQL Server licenses for ~55% savings on vCore pricing
- **Reserved instances**: 1-year (≈33% savings) or 3-year (up to ≈72% savings) commitment
- **Best for**: workloads needing fine-grained control, cost optimization, or migration from on-prem SQL Server

### DTU vs vCore Summary
| Aspect | DTU | vCore |
|---|---|---|
| Resource control | Bundled | Independent CPU/memory/storage |
| Cost transparency | Opaque (what's a DTU?) | Clear (X cores, Y GB memory) |
| Serverless | No | Yes (General Purpose + Hyperscale) |
| Azure Hybrid Benefit | No | Yes |
| Reserved instances | No | Yes |
| Hyperscale | No | Yes |
| Migration from on-prem | Harder to map | Easy to map vCores to on-prem cores |

**Recommendation**: vCore for new workloads. DTU only if simplicity is paramount and workload is small/predictable.

## Service Tiers (vCore)

### General Purpose
- Budget-friendly, balanced compute and storage
- Remote storage (Azure Premium Storage) — higher IO latency than Business Critical
- 99.99% SLA with zone redundancy
- **Serverless option**: auto-pause after configurable idle period (minimum 1 hour), auto-resume on connection. Only storage billed during pause. Billed per-second for compute used.
- Serverless supported on Standard-series (Gen5) hardware only
- **Best for**: most workloads, dev/test, applications tolerant of slightly higher IO latency

### Business Critical
- Local SSD storage — low-latency IO, built-in read replica
- 3 Always On replicas (synchronous) — one usable as free read-only endpoint
- ~2.7x price of General Purpose (paying for 3 replicas)
- 99.99% SLA (99.995% with zone redundancy)
- In-memory OLTP available for extreme OLTP performance
- **Best for**: OLTP with high IO requirements, latency-sensitive applications, need for built-in read replica

### Hyperscale
- Up to 128 TB database size (per MS Learn resource limits documentation)
- Distributed architecture with separate compute and storage components (page servers)
- Rapid scale-out of read replicas (up to 4 named replicas, 0-4 configurable)
- Near-instant backups regardless of database size (snapshot-based)
- Fast restore (minutes, not hours)
- Multi-tiered storage: local SSD cache + remote page servers
- Unlimited log size
- **Serverless available**: Hyperscale serverless on Standard-series (Gen5), auto-scales up to 80 vCores, memory up to 24 GB per vCore (dynamically adapted)
- Zone-redundant HA supported
- **Best for**: large databases, highly variable workloads, need for rapid scaling, databases that outgrow 4 TB General Purpose limit

### Service Tier Comparison
| Feature | General Purpose | Business Critical | Hyperscale |
|---|---|---|---|
| Max size | 4 TB (8 TB preview) | 4 TB (8 TB preview) | 128 TB |
| Storage type | Remote (Premium Storage) | Local SSD | Multi-tiered (local SSD + page servers) |
| IO latency | Higher (remote) | Lowest (local SSD) | Local reads: 1-2ms, Remote: 1-4ms |
| Built-in read replica | No | Yes (1 free) | Yes (0-4 named) |
| Serverless | Yes | No | Yes |
| In-memory OLTP | No | Yes | No |
| Backup speed | Size-dependent | Size-dependent | Near-instant (snapshot) |
| Zone redundancy | Yes | Yes | Yes |

## Elastic Pools
- Share resources (eDTUs or vCores) across multiple databases on same logical server
- **Ideal when**: databases have different peak times, average utilization is low but peak is high
- **Sizing formula** (DTU): MAX(total_DBs × avg_DTU, concurrent_peak_DBs × peak_DTU)
- In DTU model: pool eDTU unit price is ~1.5x single DB DTU price, but shared across many DBs = net savings
- In vCore model: same unit price as single databases
- Per-database min/max settings prevent one database from consuming all pool resources
- **SaaS pattern**: one pool per tier, databases assigned by workload intensity
- **Gotcha**: if databases all peak at the same time, elastic pool doesn't save money — it's just a shared ceiling
- **Gotcha**: Hyperscale databases cannot be placed in elastic pools

## Serverless Compute
- Auto-scales compute within configurable min/max vCore range
- Auto-pauses after idle period (configurable, minimum 1 hour) — only storage billed during pause
- Auto-resumes on first connection (cold start latency typically under 1 minute)
- Billed per-second for compute actually used
- Available in General Purpose and Hyperscale tiers
- Hyperscale serverless: up to 80 vCores, memory dynamically adapted up to 24 GB per vCore based on workload demand
- **Best for**: dev/test, intermittent workloads, applications with unpredictable usage
- **Not for**: workloads that can't tolerate cold start latency, always-on production systems with steady load
- **Cost math**: if utilization >60-70% of the time, provisioned is cheaper; below that, serverless wins
