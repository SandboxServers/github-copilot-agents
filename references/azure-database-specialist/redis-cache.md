# Azure Cache for Redis

## Use Cases
- **Session state**: offload from app servers for stateless scaling
- **Output caching**: cache rendered pages/API responses
- **Data cache**: frequently read, rarely changing data (product catalogs, config)
- **Pub/sub messaging**: lightweight real-time notifications
- **Distributed locking**: prevent concurrent access to shared resources
- **Leaderboards/counters**: sorted sets for ranking, atomic increments

## Azure Managed Redis (New — Based on Redis Enterprise)

### Architecture (Verified via Context7 + MS Learn)
- Based on Redis Enterprise — each VM/node runs multiple Redis server processes (shards) in parallel
- Clustered by default across all tiers and SKUs — clustering is a core feature, not optional
- Each SKU pre-configured with optimal shard count based on available vCPUs — not adjustable by user
- Primary and replica shards distributed across both nodes (not all primaries on same node)
- High-performance proxy process on each node handles shard management, connections, self-healing
- Reserves ~20% of memory for system operations and overhead — account for this when sizing

### Performance Tiers (Verified via Context7 + MS Learn)
| Tier | Memory:vCPU Ratio | Best For |
|---|---|---|
| **Memory Optimized** | 8:1 | Memory-intensive, lower throughput needs, dev/test |
| **Balanced** | 4:1 | Standard workloads, good starting point if unsure |
| **Compute Optimized** | 2:1 | Throughput-intensive, latency-sensitive workloads |
| **Flash Optimized** | RAM + NVMe | Large datasets, auto-moves cold data from RAM to NVMe disk |

- Scaling up is recommended before scaling out for Managed Redis
- Balanced tier has 2x vCPUs of Memory Optimized; Compute Optimized has 2x vCPUs of Balanced
- Flash Optimized: keys in DRAM perform nearly as fast as other tiers; NVMe keys are slower. Tier intelligently places frequently accessed keys in DRAM.

### Cluster Policies (Verified via Context7 + MS Learn)
| Policy | How It Works | Pros | Cons |
|---|---|---|---|
| **OSS** (recommended) | Standard Redis Cluster API — clients connect directly to shards | Lowest latency, best throughput, near-linear scaling | Requires cluster-aware client library; can't use RediSearch |
| **Enterprise** | Single endpoint, proxy routes to correct shard | No client changes needed, backward compatible | Proxy can bottleneck compute/network throughput |
| **Non-Clustered** | No sharding, ≤25 GB only | Works with any client, supports MULTI/cross-slot commands | Not as performant, limited to 25 GB, no multi-threading benefit |

- OSS clustering uses port 10000 for initial connection, 85XX range for individual nodes (don't hardcode)
- If migrating from nonclustered Basic/Standard/Premium, consider OSS clustering for better performance
- Non-Clustered only for when app absolutely can't support either OSS or Enterprise topology

### Key Differences from Azure Cache for Redis (Classic)
- No VNet injection — use Private Link instead for network isolation
- All HA/DR features available on all sizes and tiers (not tier-gated)
- HA is optional (non-HA for dev/test = cost savings, no SLA, possible data loss during maintenance)
- Microsoft Entra ID authentication supported (RBAC not yet available)
- Scaling supports changing both memory size and performance tier

## Azure Cache for Redis (Classic)

### Tiers
| Tier | Features | Best For |
|---|---|---|
| **Basic** | Single node, no SLA, no replication | Dev/test only |
| **Standard** | Replicated (primary/secondary), 99.9% SLA | Production caching |
| **Premium** | Persistence (RDB/AOF), clustering, VNet injection, geo-replication, zone redundancy | Enterprise, large datasets |
| **Enterprise** | Redis Enterprise, Redis Modules (RediSearch, RedisJSON, RedisTimeSeries, RedisBloom) | Advanced data structures, full-text search |
| **Enterprise Flash** | Enterprise on NVMe flash, 300 GB - 4.5 TB | Very large datasets with cost optimization |

- Enterprise/Enterprise Flash are always clustered internally
- Premium clustering: can enable after creation, but **cannot undo** — clustered cache with 1 shard behaves differently than non-clustered
- VNet injection and zone redundancy can only be set at Premium cache creation time, not added after

## Key Design Considerations (All Tiers)
- **Set TTL on everything** — unbounded cache grows forever until eviction pressure
- **Use connection pooling** — Redis connections are expensive to create; reuse them
- **Avoid large keys/values** — keep items under 100 KB for best performance
- **Plan for eviction** — configure maxmemory-policy (`allkeys-lru` is usually correct for caching)
- **Use TLS/SSL in production** — recommended security best practice, but decreases throughput performance
- **Single Redis client instance** — don't create new connections per request
- When scaling Standard/Premium/Enterprise to larger size, data is typically preserved. Scaling down may cause data loss if data exceeds new size (keys evicted via allkeys-lru)
