# Integration Anti-Patterns

## Architecture Anti-Patterns

| Anti-Pattern | Problem | Better Approach |
|---|---|---|
| **Synchronous chains** | Calling 5 backends in sequence causes cascading failures, compounding latency | Async messaging with Service Bus. Parallelize independent calls. |
| **Tight coupling via REST** | Direct service-to-service calls create fragile dependencies. One failure breaks the chain. | Event-driven architecture with Event Grid or Service Bus for decoupling. |
| **Polling for events** | Wasted compute, added latency, unnecessary cost from constant polling | Event Grid push model or Service Bus with Event Grid integration for reactive consumption. |
| **Event Grid for ordered delivery** | Event Grid provides no ordering guarantee. Messages may arrive out of order. | Service Bus sessions for ordered processing within a session group. |
| **APIM for internal-only APIs** | Unnecessary cost and complexity for APIs that only internal services consume | Direct service-to-service calls, or lightweight API gateway if needed. Use APIM when you need policies, developer portal, or external exposure. |
| **One giant Logic App** | Unmaintainable, hard to debug, single point of failure, long run history | Split workflows by bounded context. Use sub-workflows (call workflow action) for shared logic. |

## Reliability Anti-Patterns

| Anti-Pattern | Problem | Better Approach |
|---|---|---|
| **No dead-letter handling** | Dead letters accumulate silently, indicating broken integration nobody notices | Monitor DLQ count, alert on > 0, build automated DLQ processor. |
| **Retry without idempotency** | Retries cause duplicate processing, data corruption, double-charges | Make every consumer idempotent. Use message ID deduplication, upserts, conditional writes. |
| **No retry / no circuit breaker** | Transient failures cause immediate failure. Sustained failures cascade to callers. | Implement Polly (.NET), APIM retry policies, built-in service retries. Add circuit breaker for sustained failures. |
| **No compensation logic** | Distributed transactions partially complete with no rollback plan | Design saga pattern with compensating actions from day one. Every step needs an undo. |
| **Ignoring message ordering** | Competing consumers break ordering. Events processed out of sequence cause data inconsistency. | Use Service Bus sessions if order matters. Single consumer per session. |
| **No monitoring on queues** | Messages pile up unnoticed. Consumers crash, queues grow, DLQs fill. | Alerts on queue depth, message age, DLQ count. Dashboard for all queue metrics. |

## Cost Anti-Patterns

| Anti-Pattern | Problem | Better Approach |
|---|---|---|
| **Logic Apps for computation** | Per-action billing makes computational loops extremely expensive and slow | Azure Functions for processing logic. Logic Apps for orchestration and connector integration only. |
| **Using Premium everything** | APIM Premium ($2,794/mo), Service Bus Premium ($668/mo) when Standard suffices | Start with Standard tier. Upgrade to Premium only when you need VNet injection, AZ, multi-region, or large messages. |
| **APIM as passthrough only** | Paying for API gateway that adds no value — no policies, no auth, no caching | If not using policies, remove APIM. If keeping, add value: auth, rate limiting, caching, transformation. |
| **Consumption Logic Apps at scale** | Per-action pricing with loops over large datasets adds up fast | Switch to Logic Apps Standard for consistent high-volume workflows. Fixed monthly cost. |

## Design Anti-Patterns

| Anti-Pattern | Problem | Better Approach |
|---|---|---|
| **Chatty integration** | Many small messages/calls instead of batched operations. High overhead, high cost. | Batch messages. Aggregate events. Use claim-check for large payloads. |
| **No schema validation** | Bad messages propagate through system, cause failures at each step | Validate schemas early (at ingress). Dead-letter invalid messages immediately. |
| **Shared queues for unrelated workloads** | Different SLAs, priorities, and failure modes interfere with each other | Separate queues per workload/domain. Independent scaling and monitoring. |
| **No timeout configuration** | Default timeouts may be too long (blocking resources) or too short (failing prematurely) | Set explicit timeouts at every integration boundary. Match to expected response time + margin. |
| **Fire-and-forget without tracking** | Messages sent with no way to trace delivery or processing status | Implement distributed tracing (Application Insights). Correlate with correlation IDs across services. |

## Quick Decision Guide: When to Use What

| If You Need... | Use | NOT |
|---|---|---|
| Event notification (fan-out) | Event Grid | Service Bus |
| Reliable ordered messaging | Service Bus (sessions) | Event Grid |
| Workflow orchestration | Logic Apps | Chained HTTP calls |
| Simple compute/transforms | Functions | Logic Apps (cost) |
| API facade + policies | APIM | Custom reverse proxy |
| Data movement/ETL | Data Factory | Logic Apps |
| On-prem connectivity | On-Prem Gateway / Relay / VPN | Direct internet exposure |
