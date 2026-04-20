# Failure Design Patterns

## Retry Policies

| Service | Default Retry | Configurable |
|---|---|---|
| **Logic Apps** | Exponential (4 retries, 7.5s intervals, capped 5-45s) | Yes: None, Fixed, Exponential per action |
| **Service Bus SDK** | Exponential backoff | Yes: RetryOptions in client config |
| **Event Grid** | Exponential backoff (30 attempts, 24h) | Yes: maxDeliveryAttempts, TTL |
| **APIM** | None (fails fast by default) | Yes: `<retry>` policy in XML |
| **Functions** | Depends on trigger binding | Yes: host.json configuration |

### Retry Best Practices
- **Exponential backoff with jitter**: Prevents thundering herd on recovery. Add random jitter to avoid synchronized retries.
- **Max retries with circuit breaker**: Don't retry forever. Combine retry with circuit breaker for downstream protection.
- **Idempotent operations**: Every operation that will be retried MUST be idempotent. Safe retries require safe operations.
- **Retry only on transient failures**: 429 (throttled), 503 (unavailable), network timeouts. Don't retry 400 (bad request) or 404 (not found).

## Circuit Breaker Pattern

Prevent cascading failures by stopping calls to failing downstream services.

### States
- **Closed** (normal): Requests flow through. Failures counted.
- **Open** (failing): Requests immediately rejected (fast-fail). No calls to downstream. Timer-based transition to half-open.
- **Half-Open** (testing recovery): Allow limited requests through. If successful → Closed. If failing → Open.

### Implementation Options
- **APIM**: Backend circuit breaker configuration. Trip on 5xx count/percentage with auto-recovery window.
- **Application code (.NET)**: Polly library — `CircuitBreakerPolicy` with configurable thresholds.
- **Application code (Python)**: tenacity or pybreaker libraries.
- **Service Bus**: Natural circuit breaking via dead-letter + MaxDeliveryCount. Messages stop retrying after threshold.
- **Logic Apps**: Implement with stateful variables or external state (Redis, Table Storage) for cross-run state.

## Dead-Letter Handling

Every queue, topic, and event subscription needs dead-letter monitoring.

### Rules
- **Alert on DLQ count > 0**. Every dead-lettered message represents a failed integration.
- **Automated reprocessing**: Build a DLQ processor that inspects, fixes (if possible), and resubmits.
- **Manual investigation**: For messages that can't be auto-fixed, route to investigation queue or alert humans.
- **Retention policy**: Archive or purge old dead letters. Don't let DLQ grow unbounded.
- **Root cause analysis**: Dead letters are symptoms. Find and fix the underlying integration issue.

## Compensation (Saga Pattern)

For distributed transactions across services without distributed locks.

### Orchestration (Logic Apps / Durable Functions)
- Central coordinator directs each step and handles failures.
- Coordinator calls compensating actions when a step fails.
- Easier to understand, test, and monitor. Single point of failure.

### Choreography (Event-Driven)
- Each service emits events, next service reacts. No central coordinator.
- More resilient (no single coordinator), harder to debug and monitor.
- Event Grid + Service Bus naturally support this pattern.

### Compensation Design
- Every step must have a defined compensating action (e.g., "cancel reservation" compensates "make reservation").
- Compensating actions must also be idempotent and handle partial completion.
- Log every step and compensation for audit trail.

## Poison Messages

Messages that repeatedly fail processing despite retries.

- Delivery count increments each attempt. After `MaxDeliveryCount` → DLQ.
- **Separate poison message queue**: Isolate for investigation without blocking main processing.
- **Root causes**: Schema mismatches, corrupt data, missing dependencies, downstream permanent failures.
- **Prevention**: Validate message schema on receive. Catch and dead-letter known-bad formats explicitly.

## Idempotency

At-least-once delivery is the norm. Every receiver MUST handle duplicate messages.

| Strategy | Description | Best For |
|---|---|---|
| **Check-before-write** | DB lookup before insert | Simple CRUD operations |
| **Deduplication table** | Store processed message IDs | High-throughput messaging |
| **Natural idempotency** | Upsert instead of insert | Database writes |
| **Conditional write** | ETag / version-based updates | Concurrent access scenarios |
| **Service Bus deduplication** | MessageId-based, time window | Publisher-side duplicates |

- Service Bus duplicate detection catches publisher duplicates (time window-based).
- Consumer-side idempotency is ALWAYS the consumer's responsibility, regardless of Service Bus deduplication.

## Timeout Handling

- Set appropriate timeouts at every integration boundary. Don't use infinite timeouts.
- Circuit breaker prevents cascade from slow downstream services.
- Queue-based load leveling absorbs traffic spikes — producers enqueue, consumers process at their own rate.
- Logic Apps action timeout: configurable per action. Default varies by action type.

## Monitoring Checklist

| Metric | Alert Threshold | Service |
|---|---|---|
| Dead-letter queue depth | > 0 | Service Bus |
| Active message count (growing) | Trend increasing | Service Bus |
| Message age (oldest in queue) | > acceptable latency | Service Bus |
| Delivery failure rate | > 1% | Event Grid |
| Dead-letter events | > 0 | Event Grid |
| Failed workflow runs | > 0 | Logic Apps |
| Backend error rate (5xx) | > threshold | APIM |
| Circuit breaker state | Open | APIM / Application |
| Throttled requests | > 0 | All services |

## Troubleshooting Integration Flows

### Logic Apps
- **Run history**: Every execution with input/output per action — primary debugging tool.
- **Tracked properties**: Custom properties on action outputs for Log Analytics querying.
- **Diagnostic settings**: Send to Log Analytics for cross-workflow analysis.
- **Resubmit**: Replay failed runs from portal (Consumption) or trigger history (Standard).

### Service Bus
- **Service Bus Explorer** (portal): Peek, receive, send, dead-letter management.
- **Application Insights**: SDK telemetry for send/receive with distributed tracing.
- **Azure Monitor metrics**: Active messages, dead-letter count, incoming/outgoing, server errors.

### Event Grid
- **Delivery metrics**: Success/failure counts per subscription.
- **Dead-letter storage**: Inspect undelivered events in blob container.
- **Diagnostic logs**: Publish/delivery failures with error details.

### APIM
- **Request tracing**: `Ocp-Apim-Trace: true` header for detailed policy execution trace.
- **Application Insights**: Full request/response logging, dependency tracking.
- **Built-in analytics**: Dashboard for usage, response times, failures.
- **Diagnostic logs**: Gateway logs to Log Analytics.
