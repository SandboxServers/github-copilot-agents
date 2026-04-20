# Cosmos DB

## Core Concepts
- Globally distributed, multi-model database service
- APIs: NoSQL (native, recommended for new apps), MongoDB, PostgreSQL, Cassandra, Gremlin, Table
- Turnkey global distribution — add/remove regions with a click
- Single-digit millisecond reads and writes at any scale
- 99.999% SLA with multi-region writes

## Request Units (RUs)

### Fundamentals
- RU = abstraction of CPU, IOPS, memory for an operation
- 1 RU = 1 point read of 1 KB item (by ID and partition key)
- 1 KB write = ~5 RUs, 100 KB write = ~50 RUs
- Query cost varies by item count, predicate complexity, result set size, index usage
- Strong/Bounded Staleness consistency doubles read RU cost (two-replica reads)

### Multi-Region RU Math
- Container configured with R RU/s across N regions = R × N total RU/s billed
- Adding a region multiplies the RU bill — this is the single biggest cost trap
- Session/Eventual consistency provides ~2x read throughput vs Strong/Bounded Staleness

### Provisioning Modes

| Mode | Billing | Best For |
|---|---|---|
| Provisioned | Per RU/s per hour | Predictable, steady workloads |
| Autoscale | Scales 10-100% of max RU/s, billed at peak per hour | Variable but predictable range |
| Serverless | Per RU consumed (pay-per-request) | Sporadic, dev/test, low volume |

### RU Optimization
1. Use point reads (ID + partition key) whenever possible — cheapest at 1 RU per 1 KB
2. Choose weaker consistency if possible — Session is the sweet spot (1x RU cost)
3. Optimize indexing policy — exclude unused paths, use composite indexes for ORDER BY
4. Reduce item size — fewer properties = fewer RUs for writes
5. Use autoscale — prevents 429 throttling while minimizing waste (min bill = 10% of max)
6. Monitor 429 responses indicating under-provisioning

## Partition Key Design — Most Critical Decision

### Fundamentals
- Logical partition: all items with same partition key value (max 20 GB)
- Physical partition: one or more logical partitions served by same replica set
- Good partition key: high cardinality, even distribution of storage AND throughput
- Bad partition key: date (hot partition for today), status enum (few values), boolean

### Guidelines
1. Pick a property present in most queries (avoids expensive cross-partition fan-out)
2. High cardinality — hundreds or thousands of unique values minimum
3. Even distribution — no single value should dominate storage or request volume
4. Synthetic partition key: combine properties if no single one works (e.g., userId-year)
5. Cannot be changed after creation — must recreate container

### Hierarchical Partition Keys
- Up to 3 levels of hierarchy for partition keys (subpartitioning)
- Logical partition prefixes can exceed 20 GB and 10,000 RU/s with hierarchical keys
- Example: /tenantId → /tenantId/userId → /tenantId/userId/sessionId
- Queries by prefix efficiently routed to subset of partitions

## Consistency Levels

| Level | Guarantee | RU Cost | Use Case |
|---|---|---|---|
| Strong | Linearizable reads | 2x | Financial transactions, inventory |
| Bounded Staleness | Reads lag by K versions or T time | 2x | Leaderboards, near-real-time analytics |
| Session (default) | Read-your-writes within session | 1x | Most applications — best default |
| Consistent Prefix | Reads never see out-of-order writes | 1x | Social feeds, event sequences |
| Eventual | No ordering guarantee | 1x | Counts, non-critical reads |

Cost decreases as consistency weakens. Session is the right choice for 90% of applications.

### Critical Rules
- Strong consistency NOT available with multi-region writes — falls back to Bounded Staleness
- Consistency can be relaxed per-request (account default Session, specific query uses Eventual)
- Session token: client receives updated token after each write, sends on reads

## Global Distribution
- Multi-region writes (active-active) with automatic conflict resolution
- Last Writer Wins (default), custom merge procedures, or async conflict resolution
- Single-region write + multi-region read with automatic failover
- Service-managed failover (automatic) or manual — configurable per account

### RPO by Configuration

| Regions | Write Mode | Consistency | RPO |
|---|---|---|---|
| 1 | Single | Any | < 240 minutes |
| >1 | Single write | Session/Prefix/Eventual | < 15 minutes |
| >1 | Single write | Bounded Staleness | K & T (staleness window) |
| >1 | Single write | Strong | 0 |
| >1 | Multi write | Session/Prefix/Eventual | < 15 minutes |

## Pricing
- RU/s × hours + storage (GB/month)
- Multi-region multiplies RU cost proportionally (R × N regions)
- Reserved capacity for 1-3 year discounts on provisioned throughput
- Free tier available: 1,000 RU/s + 25 GB storage

## Vector Search (MongoDB vCore)
- Integrated vector database — embeddings stored alongside operational data
- Index types: DiskANN (recommended for >1M docs), HNSW (M30+ tiers), IVF (general purpose)
- Eliminates need for separate vector database — reduces cost and complexity
