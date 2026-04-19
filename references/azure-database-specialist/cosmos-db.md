# Cosmos DB

## Core Concepts
- Globally distributed, multi-model database
- APIs: NoSQL (native), MongoDB, Cassandra, Gremlin (graph), Table, PostgreSQL
- Turnkey global distribution — add/remove regions with a click
- Single-digit millisecond reads/writes at any scale
- 99.999% SLA with multi-region writes
- Integrated vector database in MongoDB vCore — vector search alongside operational data without separate vector DB

## Request Units (RUs) — The Currency That Bankrupts People

### RU Fundamentals
- RU = abstraction of CPU, IOPS, memory for an operation
- **1 RU ≈ 1 point read of 1 KB item** (by ID and partition key)
- 1 KB write ≈ 5 RUs, 100 KB write ≈ 50 RUs
- Query cost varies: depends on item count, predicate complexity, result set size, index usage
- **Strong/Bounded Staleness consistency doubles read RU cost** — reads are done against two replicas (minority quorum in a four-replica set) vs single-replica reads for Session/Eventual (verified via MS Learn consistency documentation)

### RU Math for Multi-Region (Verified via MS Learn)
- If container is configured with R RU/s and there are N regions: total RU/s available globally = R × N
- You are billed for R × N total RU/s
- Consistency level affects throughput: ~2x read throughput for Session/Consistent Prefix/Eventual vs Strong/Bounded Staleness
- **This is the single biggest cost trap** — forgetting that adding a region multiplies your RU bill

### Provisioning Modes
| Mode | Billing | Best For |
|---|---|---|
| **Provisioned** | Per RU/s per hour | Predictable, steady workloads |
| **Autoscale** | Scales 10-100% of max RU/s, billed at peak in each hour | Variable but predictable range |
| **Serverless** | Per RU consumed (pay-per-request) | Sporadic, dev/test, low volume |

### RU Cost Optimization
1. **Use point reads** (ID + partition key) whenever possible — cheapest operation (1 RU per 1 KB)
2. **Choose weaker consistency** if possible — Session is the sweet spot (read-your-writes guarantee, 1x RU cost)
3. **Optimize indexing policy** — exclude unused paths, use composite indexes for ORDER BY queries
4. **Reduce item size** — fewer properties = fewer RUs for writes
5. **Use the Capacity Calculator** (cosmos.azure.com/capacitycalculator) — model workload before provisioning
6. **Monitor with metrics** — watch for 429 (throttled) responses indicating under-provisioning
7. **Consider autoscale** — prevents 429s by scaling to demand, minimum bill is 10% of max RU/s

## Partition Key Design — Make or Break Decision

### Fundamentals
- **Logical partition**: all items with same partition key value (max 20 GB per logical partition)
- **Physical partition**: one or more logical partitions served by same replica set
- **Partition-set**: group of physical partitions across regions managing the same key range (geo-distributed overlay)
- **Good partition key**: high cardinality, even distribution of storage AND throughput
- **Bad partition key**: date (hot partition for today), status enum (few values), boolean

### Partition Key Guidelines
1. Pick a property present in most queries (avoids expensive cross-partition fan-out queries)
2. High cardinality — hundreds/thousands of unique values minimum
3. Even distribution — no single value should dominate storage or request volume
4. **Synthetic partition key**: combine multiple properties if no single property works (e.g., `userId-year`)
5. **Test with representative data** — partition key can't be changed after creation (must recreate container)

### Hierarchical Partition Keys (Verified via MS Learn)
- Cosmos DB supports up to 3 levels of hierarchy for partition keys (subpartitioning)
- Logical partition key prefixes can exceed 20 GB and 10,000 RU/s with hierarchical keys
- Useful when: synthetic keys are used today, partition keys can exceed 20 GB, or you want each tenant to map to its own logical partition
- Queries by prefix are efficiently routed to the subset of partitions holding the data
- Example: `/tenantId` → `/tenantId/userId` → `/tenantId/userId/sessionId`

## Consistency Levels (Verified via MS Learn)

| Level | Guarantee | Quorum Reads | Quorum Writes | RU Cost | Use Case |
|---|---|---|---|---|---|
| **Strong** | Linearizable reads | Local Minority (2 replicas) | Global Majority | 2x | Financial transactions, inventory |
| **Bounded Staleness** | Reads lag by ≤ K versions or T time | Local Minority (2 replicas) | Local Majority | 2x | Leaderboards, near-real-time analytics |
| **Session** (default) | Read-your-writes within session | Single Replica (session token) | Local Majority | 1x | Most applications — best default |
| **Consistent Prefix** | Reads never see out-of-order writes | Single Replica | Local Majority | 1x | Social feeds, event sequences |
| **Eventual** | No ordering guarantee | Single Replica | Local Majority | 1x | Counts, non-critical reads |

### Consistency & Durability — RPO by Configuration (Verified via MS Learn)
| Regions | Write Mode | Consistency | RPO |
|---|---|---|---|
| 1 | Single or Multi | Any | < 240 Minutes |
| >1 | Single write | Session/Prefix/Eventual | < 15 minutes |
| >1 | Single write | Bounded Staleness | K & T (staleness window) |
| >1 | Single write | Strong | 0 |
| >1 | Multi write | Session/Prefix/Eventual | < 15 minutes |
| >1 | Multi write | Bounded Staleness | K & T |

### Critical Consistency Rules
- **Strong consistency NOT available with multi-region writes** — falls back to Bounded Staleness
- Strong consistency with >5,000 miles (8,000 km) between regions is blocked by default — requires support ticket
- **Session** is the right choice for 90% of applications — read-your-writes guarantee with 1x RU cost
- Consistency can be relaxed per-request (e.g., account default Session, specific query uses Eventual)
- Session token: client receives updated token after each write, sends it on reads. If replica doesn't have data for token, retries against other replicas. Always returns latest data even with outdated token.
- Write latency for Strong: 2× round-trip time between farthest regions + 10ms at p99

## Vector Search (Cosmos DB for MongoDB vCore)
- Integrated vector database — embeddings stored alongside operational data
- Index types: DiskANN (recommended for >1M docs), HNSW (M30+ tiers), IVF (general purpose)
- DiskANN supports combined vector and geospatial queries
- HNSW parameters: `m` (max connections per layer, default 16), `efConstruction` (candidate list size, default 64)
- IVF `numLists`: set to `documentCount/1000` for ≤1M docs, `sqrt(documentCount)` for >1M docs
- Eliminates need for separate vector database — reduces cost, improves consistency
