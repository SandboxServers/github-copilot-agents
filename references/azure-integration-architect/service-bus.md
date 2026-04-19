## Service Bus

### Tiers
| Feature | Basic | Standard | Premium |
|---|---|---|---|
| **Queues** | Yes | Yes | Yes |
| **Topics/Subscriptions** | No | Yes | Yes |
| **Sessions (FIFO)** | No | Yes | Yes |
| **Transactions** | No | Yes | Yes |
| **Duplicate detection** | No | Yes | Yes |
| **Partitioning** | Yes (16 fixed) | Yes (16 fixed) | Yes (configurable at namespace creation) |
| **Max message size** | 256 KB | 256 KB | 100 MB |
| **Namespace size** | 400 GB | 400 GB | 1 TB per messaging unit |
| **Throughput** | Variable, subject to throttling | Variable, subject to throttling | Predictable, per messaging unit (~4 MB/s per MU) |
| **Pricing** | Per operation | Per operation + base | Per messaging unit (fixed) |
| **VNet/Private endpoint** | No | No | Yes |
| **Geo-DR** | No | Replication | Full geo-disaster recovery |
| **Auto-scale** | No | No | Yes (messaging units) |

### Key Decision: Standard vs Premium
- **Standard**: adequate for most dev/test and moderate production. Per-operation pricing. Subject to noisy-neighbor throttling.
- **Premium**: required for production workloads needing predictable latency, large messages (>256 KB), VNet isolation, sessions + partitioning together, or geo-DR. Fixed-cost per MU.
- **Basic**: only for simple queue scenarios without topics. Don't use in production unless cost is extreme constraint.

### Messaging Patterns

#### Queue (Point-to-Point)
- One sender, one receiver per message
- Messages persist until consumed or expired
- **Competing consumers**: multiple receivers pull from same queue — load balancing + resilience
- Use **PeekLock** mode: receive → process → Complete (or Abandon/DeadLetter)
- **ReceiveAndDelete**: lower latency but message lost if processing fails

#### Topics & Subscriptions (Pub/Sub)
- One publisher, multiple subscribers — each subscription gets a copy
- Subscriptions can have **filters** (SQL filter, correlation filter) to receive subset of messages
- Use for broadcasting events to multiple consumers
- Topic granularity: broad topics + subscription filters vs narrow topics — balance management vs filtering overhead

#### Request/Reply
- Use **message sessions** with `ReplyToSessionId` for correlated request-response
- Sender sets `ReplyTo` queue and `SessionId` — receiver replies to that session
- Enables ordered, correlated bidirectional communication

#### Dead Letter Queue (DLQ)
- Every queue/subscription has a DLQ subqueue
- Messages moved to DLQ when:
  - **MaxDeliveryCount exceeded** (default: 10) — poison message protection
  - **TTL expired** — message sat too long without being consumed
  - **Application-level dead-lettering** — consumer explicitly dead-letters unprocessable messages
  - **Auto-forward chain > 4 hops**
  - **Destination disabled or at capacity**
- **Always monitor DLQ** — unprocessed dead letters indicate integration failures
- Set `DeadLetterReason` and `DeadLetterDescription` when explicitly dead-lettering for troubleshooting

#### Poison Message Handling
- A message that repeatedly fails processing → delivery count increments each attempt
- After `MaxDeliveryCount` (configurable), automatically moved to DLQ
- **Best practice**: set `MaxDeliveryCount` based on expected transient failure duration (3-5 for fast failures, 10+ for retryable)
- **Monitor DLQ depth** as a key operational metric — growing DLQ means broken integration
- Create a DLQ processor to inspect, fix, and resubmit or archive dead-lettered messages

### Sessions & Ordering
- **Sessions**: guarantee FIFO within a session (messages with same `SessionId` processed in order)
- Session state: consumer can store checkpoint state with `SetSessionStateAsync` — enables long-running transaction coordination
- **Gotcha**: sessions are immutable after entity creation — must enable at queue/subscription creation time
- **Gotcha**: partitioning + sessions in Standard tier not recommended; Premium handles both properly
