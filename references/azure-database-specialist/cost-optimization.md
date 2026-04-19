# Cost Optimization

## General Strategies

### Right-Sizing
- Don't over-provision — use Azure Monitor metrics to find actual CPU, IO, memory utilization
- If utilization is consistently <30%, you're overpaying — scale down
- Use Azure Advisor recommendations for right-sizing suggestions
- Review monthly and after major workload changes

### Commitment Discounts
- **Reserved instances**: 1-year (≈33% savings) or 3-year (up to ≈72% savings) for predictable workloads
- **Azure Hybrid Benefit**: bring existing SQL Server licenses for ~55% savings on vCore pricing
- Both stack: Reserved + AHB can reduce costs by 80%+ vs pay-as-you-go
- **Caveat**: reserved capacity is use-it-or-lose-it — only commit what you'll consistently use

### Compute Cost Reduction
- **Serverless** (SQL Database): for intermittent workloads — only pay for compute when active, auto-pause saves 100% compute during idle
- **Stop/Start** (PostgreSQL and MySQL Flexible Server): manually stop in dev/test — no compute charges while stopped
- **License-free standby**: DR-only geo-replicas (SQL Database) don't need to pay for SQL Server license
- **Elastic pools**: share vCores/eDTUs across databases with staggered peak usage patterns

### Monitoring & Alerting
- Set up Azure cost alerts at subscription and resource group level
- Use Azure Advisor cost recommendations — regularly review
- Tag databases by team/project/environment for cost allocation and chargeback
- Review cost trends monthly — catch creep before it compounds

## Azure SQL Database Cost Specifics

### Serverless Economics
- Billed per-second for compute used
- Auto-pause: only storage billed (configurable idle timeout, minimum 1 hour)
- **Break-even**: if utilization >60-70%, provisioned is cheaper. Below that, serverless wins.
- Cold start penalty: ~1 minute on resume — acceptable for dev/test, potentially problematic for production

### Elastic Pool Economics
- DTU model: pool eDTU unit price is ~1.5x single DB, but shared across many DBs = net savings when utilization is staggered
- vCore model: same unit price as single databases — savings come from sharing
- **Sizing**: MAX(total_DBs × avg_utilization, concurrent_peak_DBs × peak_utilization)
- **Gotcha**: if all databases peak simultaneously, pool doesn't save money

### Business Critical vs General Purpose
- Business Critical is ~2.7x the cost of General Purpose (3 synchronous replicas)
- Only choose BC if you genuinely need: local SSD latency, built-in read replica, or in-memory OLTP
- Many workloads that "feel" like they need BC actually work fine on General Purpose with proper indexing

### Hyperscale Cost Model
- Charged per vCore (each replica) + used storage (not reserved storage)
- Named replicas billed at same vCore rate — add only what you need
- Snapshot-based backups mean less backup storage churn than traditional tiers

## Cosmos DB Cost Traps

### The Big Ones
1. **Over-provisioning RU/s** — start low, use autoscale (scales 10-100% of max, billed at peak per hour), monitor 429 throttle responses
2. **Multi-region RU multiplication** — R RU/s × N regions = total RU/s billed. Adding a region doubles your bill (for 2-region). People forget this.
3. **Cross-partition queries** — fan out to all partitions, consume more RUs than targeted queries. Fix with better data modeling and partition key design.
4. **Strong/Bounded Staleness consistency** — 2x RU cost for reads vs Session. Switch to Session unless you genuinely need linearizable reads.
5. **Indexing everything by default** — default policy indexes all properties. Exclude write-heavy paths you never query to reduce write RU cost.
6. **Large items** — every byte costs RUs on writes. Keep documents lean — reference large blobs in Blob Storage instead.
7. **Serverless for always-on workloads** — serverless has per-RU pricing that's higher than provisioned. It's only cheaper for sporadic, low-volume workloads.

### Cosmos DB Cost Optimization Checklist
- [ ] Use point reads (ID + partition key) wherever possible — 1 RU per 1 KB
- [ ] Default to Session consistency — read-your-writes at 1x RU cost
- [ ] Exclude unused index paths — reduce write RU overhead
- [ ] Use autoscale instead of fixed provisioned — prevents 429s while minimizing waste
- [ ] Monitor per-partition metrics — uneven distribution = wasted provisioned RU/s
- [ ] Consider Cosmos DB free tier for dev/test (1,000 RU/s + 25 GB free)
- [ ] Review multi-region topology — do you need writes in all regions, or just reads?

## PostgreSQL & MySQL Cost Tips
- **Stop/Start**: stop Flexible Server in dev/test outside business hours — zero compute while stopped
- **Right-size compute**: Burstable SKUs (B-series) for dev/test, General Purpose (D-series) for production
- **Storage auto-grow**: enabled by default — monitor to avoid surprise storage bills
- **HA doubles compute cost**: only enable zone-redundant HA for production workloads that need 99.99% SLA

## Redis Cost Tips
- **Azure Managed Redis non-HA option**: for dev/test, disable HA to halve cost (no SLA, possible data loss)
- **Right-size memory**: Azure Managed Redis reserves ~20% for system overhead. A 12 GB SKU provides ~9.6 GB usable.
- **Review Used Memory %**: if actual usage is well below cache size, downsize to smaller SKU
- **Don't over-persist**: persistence (RDB/AOF) adds IO overhead — only enable if data must survive full restart
- **TTL everything**: unbounded caches grow until eviction pressure forces costly scaling
