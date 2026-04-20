# Database Cost Optimization

## SQL Database

### Serverless
- Auto-pause after configurable idle period — only storage billed during pause
- Billed per-second for compute actually used
- Break-even: if utilization above 60-70%, provisioned is cheaper; below that, serverless wins
- Cold start ~1 minute on resume — acceptable for dev/test

### Elastic Pools
- Share vCores/eDTUs across databases with staggered peak usage patterns
- DTU model: pool price ~1.5x single DB, shared across many = net savings
- Gotcha: if all databases peak simultaneously, pool does not save money

### Reserved Capacity
- 1-year (~33% savings) or 3-year (up to ~72% savings) for predictable workloads
- Azure Hybrid Benefit: bring SQL Server licenses for ~55% savings on vCore
- Both stack: Reserved + AHB can reduce costs by 80%+ vs pay-as-you-go

### Tier Right-Sizing
- Business Critical is ~2.7x General Purpose cost — only if you need local SSD, read replica, or In-Memory OLTP
- Many workloads that feel like they need BC work fine on GP with proper indexing
- DTU to vCore migration for right-sizing with transparent resource control
- Hyperscale for better price/performance at scale with snapshot backups

## Cosmos DB

### Top Cost Traps
1. Over-provisioning RU/s — use autoscale (10-100% of max, billed at peak per hour)
2. Multi-region RU multiplication — R RU/s × N regions = total bill; adding a region doubles cost
3. Cross-partition queries — fan out to all partitions, consuming more RUs than targeted queries
4. Strong/Bounded Staleness consistency — 2x RU for reads vs Session
5. Default indexing policy indexes everything — exclude write-heavy paths never queried
6. Large items — every byte costs RUs on writes; reference blobs in Blob Storage instead
7. Serverless for always-on workloads — per-RU pricing higher than provisioned

### Optimization Checklist
- Use point reads (ID + partition key) wherever possible — 1 RU per 1 KB
- Default to Session consistency — read-your-writes at 1x RU cost
- Exclude unused index paths to reduce write RU overhead
- Use autoscale instead of fixed provisioned to prevent 429s while minimizing waste
- Monitor per-partition metrics — uneven distribution wastes provisioned RU/s
- Free tier for dev/test (1,000 RU/s + 25 GB free)
- Reserved capacity for steady production (1-3 year discounts)

## PostgreSQL & MySQL
- Burstable SKUs (B-series) for dev/test, General Purpose (D-series) for production
- Stop/Start: stop Flexible Server outside business hours — zero compute while stopped
- HA doubles compute cost — only enable zone-redundant HA for production needing 99.99% SLA
- Reserved capacity for predictable production workloads
- Storage auto-grow enabled by default — monitor to avoid surprise bills

## Redis
- Basic tier for dev only (no SLA, no replication)
- Standard for small production workloads
- Enterprise Flash for large datasets (cheaper than Enterprise for same memory)
- Set TTL on all keys — unbounded caches grow until eviction forces costly scaling
- Right-size memory based on actual Used Memory percentage

## Common Waste Patterns
- Over-provisioned DTUs/RUs with consistently low utilization
- Premium/Business Critical tier for development databases
- Geo-replication enabled in dev/test environments
- Unused read replicas still incurring compute cost
- No auto-pause configured on serverless databases
- Paying full price without reserved capacity for steady-state production

## Monitoring
- Set Azure cost alerts at subscription and resource group level
- Review Azure Advisor cost recommendations regularly
- Tag databases by team/project/environment for cost allocation
- Review cost trends monthly to catch spending creep early
