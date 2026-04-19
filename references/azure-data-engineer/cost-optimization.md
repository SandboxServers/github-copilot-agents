# Cost Optimization

## The Expensive Mistakes

| Trap | Why It Happens | Monthly Impact | Fix |
|------|---------------|----------------|-----|
| Synapse dedicated pool always on | Default is running. No auto-pause. | $5K-$50K+ | Pause when not in use. Or migrate to serverless/Fabric. |
| Databricks clusters never terminate | Auto-terminate disabled or set too long | $3K-$20K+ | Set auto-terminate to 10-15min. Use cluster policies. |
| Storage transactions > storage cost | Millions of small files, frequent listing | $1K-$10K+ | Compact files, use Delta, partition wisely |
| Over-provisioned Fabric capacity | Bought F64 but using F16 worth | $5K+/mo waste | Start small, monitor CU utilization, scale up when data demands |
| Databricks Photon on wrong workloads | Photon adds cost but not all workloads benefit | 2x compute cost | Enable only for SQL-heavy workloads, not streaming/ML |
| Premium storage tier for cold data | All data in Hot tier by default | 2-3x storage cost | Lifecycle policies: hot → cool → archive based on access |
| Synapse serverless scanning full partitions | No partition pruning, scanning TB per query | $1K+ per TB scanned | Partition by date, use Delta statistics, limit scan scope |

## Cost Optimization Levers

- **Fabric**: Right-size capacity. Pause when not needed. Use Fabric trial/dev capacities for non-prod.
- **Databricks**: Spot instances for batch, auto-terminate for interactive, cluster pools for fast startup, photon only where it helps.
- **Storage**: Lifecycle policies, ADLS access tiers, compact small files (reduce transaction count), Delta Z-order for query pruning.
- **Synapse**: Pause dedicated pools (or migrate away). Use serverless for ad-hoc. Consider Fabric as replacement.
