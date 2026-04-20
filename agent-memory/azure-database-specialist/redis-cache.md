# Azure Cache for Redis

## Use Cases
- Session state: offload from app servers for stateless scaling
- Output caching: cache rendered pages and API responses
- Distributed locking: prevent concurrent access to shared resources
- Real-time leaderboards: sorted sets for ranking, atomic increments
- Rate limiting: sliding window counters for API throttling
- Pub/sub messaging: lightweight real-time notifications

## Tiers

| Tier | Features | SLA | Best For |
|---|---|---|---|
| Basic | Single node, no replication | None | Dev/test only |
| Standard | Replicated primary/secondary | 99.9% | Production caching |
| Premium | Clustering, persistence, VNet, zones | 99.9% | Enterprise, large datasets |
| Enterprise | Redis modules, active geo-replication | 99.9% | Advanced data structures, full-text search |
| Enterprise Flash | Enterprise on NVMe flash, 300GB-4.5TB | 99.9% | Very large datasets, cost-effective |

- Enterprise/Enterprise Flash are always clustered internally
- Premium clustering: can enable after creation but cannot undo
- VNet injection and zone redundancy only at Premium creation time

## Sizing Guidelines
- Start with C1 Standard (~$40/month) for small production workloads
- Scale up based on memory needs and connection count
- Premium for datasets >53GB or when clustering is needed
- Enterprise for Redis modules (RediSearch, RedisJSON, RedisTimeSeries, RedisBloom)
- Enterprise Flash for large datasets needing cost optimization over pure Enterprise

## Best Practices
- Use connection multiplexing — Redis connections are expensive to create; reuse them
- Set TTL on all keys — unbounded cache grows forever until eviction pressure
- Keep items under 100 KB for best performance
- Configure maxmemory-policy (allkeys-lru is usually correct for caching)
- Use TLS/SSL in production for security
- Single Redis client instance per application — do not create new connections per request
- Use Redis Streams for messaging patterns
- Enable Entra ID authentication on Enterprise tier
- Private endpoints for production network isolation

## Data Persistence
- RDB snapshots: periodic point-in-time snapshots (Premium+ only)
- AOF (Append Only File): logs every write operation (Premium+ only)
- For recovery scenarios, not primary storage — Redis is a cache
- Import/export available for migration between caches

## Scaling Considerations
- Scaling up Standard/Premium preserves data (typically)
- Scaling down may cause data loss if data exceeds new size (keys evicted via allkeys-lru)
- When scaling, plan for brief connectivity interruption
- Premium clustering enables horizontal scale-out across shards
