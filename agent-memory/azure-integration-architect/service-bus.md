# Service Bus

## Tiers

| Feature | Basic | Standard | Premium |
|---|---|---|---|
| **Queues** | Yes | Yes | Yes |
| **Topics/Subscriptions** | No | Yes | Yes |
| **Sessions (FIFO)** | No | Yes | Yes |
| **Transactions** | No | Yes | Yes |
| **Duplicate detection** | No | Yes | Yes |
| **Max message size** | 256 KB | 256 KB | 100 MB |
| **Namespace storage** | 400 GB | 400 GB | 1 TB per messaging unit |
| **VNet/Private endpoint** | No | No | Yes |
| **Availability zones** | No | No | Yes |
| **Geo-DR** | No | Metadata replication | Full geo-disaster recovery |
| **Auto-scale** | No | No | Yes (messaging units) |
| **Pricing** | Per operation | Per operation + base | Per messaging unit (~$668/MU/month) |

## Tier Selection

- **Basic**: Queues only. Don't use in production unless cost is extreme constraint. No topics, sessions, or transactions.
- **Standard**: Queues + topics. Adequate for dev/test and moderate production. Per-operation pricing. Subject to noisy-neighbor throttling on shared infrastructure.
- **Premium**: Required for production needing predictable latency, large messages (>256 KB), VNet isolation, availability zones, or geo-DR. Fixed-cost per messaging unit. Start with 1 MU (~4 MB/s throughput).

## Entities and Messaging Patterns

### Queues (Point-to-Point)
- One sender, one receiver per message. Messages persist until consumed or expired.
- **Competing consumers**: Multiple receivers pull from same queue — provides load balancing and resilience.
- Use **PeekLock** mode (preferred): Receive → Process → Complete (or Abandon/DeadLetter).
- **ReceiveAndDelete**: Lower latency but message lost if processing fails. Only for non-critical data.

### Topics and Subscriptions (Pub/Sub)
- One publisher, multiple subscribers. Each subscription gets a copy of matching messages.
- **Filter types**: SQL filter (SQL-like expressions on properties), Correlation filter (exact match on properties, most performant), Boolean filter (true = all, false = none).
- Topic granularity: Broad topics + subscription filters vs narrow topics. Balance management overhead vs filtering complexity.
- Each subscription is an independent queue — subscribers process at their own pace.

### Sessions (Ordered Processing)
- Guarantee FIFO within a session. Messages with same `SessionId` processed in order by one consumer.
- Session state: Consumer stores checkpoint with `SetSessionStateAsync` — enables long-running transaction coordination.
- **CRITICAL**: Sessions must be enabled at queue/subscription creation time. Cannot be changed after.
- **CRITICAL**: Partitioning + sessions in Standard tier not recommended. Premium handles both properly.

## Dead-Letter Queue (DLQ)

Every queue and subscription has a DLQ subqueue. Messages move to DLQ when:
- **MaxDeliveryCount exceeded** (default 10) — poison message protection
- **TTL expired** — message sat too long without being consumed
- **Application dead-lettering** — consumer explicitly dead-letters unprocessable messages
- **Filter evaluation exception** — subscription filter throws error
- **Auto-forward chain > 4 hops** or destination entity disabled/at capacity

### DLQ Best Practices
- **ALWAYS monitor DLQ depth** — growing DLQ means broken integration
- Alert on DLQ count > 0. Investigate immediately.
- Set `DeadLetterReason` and `DeadLetterDescription` when explicitly dead-lettering for troubleshooting
- Create a DLQ processor to inspect, fix, and resubmit or archive dead-lettered messages
- Set `MaxDeliveryCount` based on expected transient failure duration (3-5 for fast failures, 10+ for retryable)

## Key Features

### Message Deduplication
- Time window-based using `MessageId`. Catches publisher duplicates within the deduplication window.
- Enable at queue/topic creation. Set deduplication window (default 10 minutes, max 7 days).
- Does NOT prevent consumer-side duplicates — consumers still need idempotency.

### Scheduled Delivery
- Set `ScheduledEnqueueTimeUtc` to delay message availability. Useful for delayed processing, reminders, timed workflows.

### Auto-Forwarding
- Automatically forward messages from one queue/subscription to another entity. Chain up to 4 hops.
- Use for fan-out patterns or routing architectures.

### Transactions
- Group send/complete operations in a transaction scope. All succeed or all fail.
- Scoped to a single namespace. Cannot span multiple namespaces or other Azure services.

## Security

- **Managed identity** (preferred): Assign Azure RBAC roles (Service Bus Data Sender, Receiver, Owner).
- **Shared Access Policies** (legacy): Connection strings with send/listen/manage rights. Rotate regularly.
- **Private endpoints**: Premium tier only. Service Bus accessible only via private IP on your VNet.
- **Encryption**: At rest (Microsoft-managed or customer-managed keys in Premium). In transit (TLS 1.2).

## Monitoring

- **Key metrics**: Active messages, dead-letter count, incoming/outgoing messages, server errors, throttled requests.
- **Alerts**: DLQ count > 0, active message count growing (consumers not keeping up), server errors.
- **Service Bus Explorer** (portal): Peek, receive, send, dead-letter management.
- **Application Insights**: SDK telemetry for send/receive operations with distributed tracing.
