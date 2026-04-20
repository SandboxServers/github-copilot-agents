# Data Platform Cost Optimization

Strategies to control and reduce data platform spending.

## Fabric Cost Management

- Capacity-based pricing with F SKUs: pay for reserved compute capacity
- F2 for development at ~$262/month — sufficient for individual dev/test
- Pause capacity when not in use (nights, weekends) to stop billing
- Auto-scale capacity if workloads have unpredictable peaks
- Smoothing and bursting: Fabric handles short spikes without scaling up
- Use Fabric trial or dev capacities for non-production environments
- Monitor CU utilization — if consistently under 50%, downsize the SKU
- Right-size by workload: don't buy F64 when F16 covers actual usage

## Databricks Cost Management

- DBU-based pricing varies by cluster type (jobs compute is cheaper than all-purpose)
- Spot instances for worker nodes: 60-90% savings, acceptable for batch workloads
- Cluster auto-termination: set to 10-15 minutes to avoid idle costs
- Cluster pools: pre-allocated VMs for faster startup without always-on costs
- Photon engine: enables faster SQL but costs more — enable only for SQL-heavy workloads
- Reserved capacity: commit to 1-3 year DBU reservations for predictable workloads
- Use jobs clusters for scheduled ETL (spin up, run, terminate) instead of interactive

## Storage Cost Optimization

- ADLS Gen2 access tiers: hot (frequent), cool (infrequent), archive (rare)
- Lifecycle management policies: automatically move data between tiers based on age
- Delta Lake VACUUM: reclaim storage from old file versions regularly
- Compression: Parquet/Delta default compression reduces storage 50-80%
- Compact small files with OPTIMIZE: fewer files = fewer transaction costs
- Delete or archive bronze data beyond retention requirements
- Monitor storage transactions — can exceed storage cost with many small files

## Compute Right-Sizing

- Monitor Spark cluster utilization: CPU, memory, shuffle read/write
- Auto-scaling clusters: set min/max nodes based on workload patterns
- Photon acceleration for Databricks SQL workloads (not beneficial for streaming/ML)
- Right-size node types: memory-optimized for joins, compute-optimized for transforms
- Serverless SQL in Synapse for ad-hoc queries instead of dedicated pools

## Common Waste Patterns

- Idle Databricks clusters with auto-terminate disabled: $3K-$20K+/month
- Synapse dedicated pools running 24/7 without pausing: $5K-$50K+/month
- Over-provisioned Fabric capacity (bought F64, using F16 worth): $5K+/month waste
- All data in hot storage tier when most is rarely accessed: 2-3x storage cost
- No VACUUM policy: unbounded storage growth from Delta file versions
- Unused Power BI datasets still consuming capacity resources
- Storage transactions exceeding storage cost due to millions of small files
