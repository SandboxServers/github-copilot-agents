## Designing for Failure

### Retry Policies
| Service | Default Retry | Configurable |
|---|---|---|
| **Logic Apps** | Exponential (4 retries, 7.5s intervals, capped 5-45s) | Yes: None, Fixed, Exponential per action |
| **Service Bus SDK** | Exponential backoff | Yes: RetryOptions in client config |
| **Event Grid** | Exponential backoff (30 attempts, 24h) | Yes: maxDeliveryAttempts, TTL |
| **APIM** | None (fails fast) | Yes: `<retry>` policy in XML |
| **Functions** | Depends on trigger binding | Yes: host.json configuration |

### Circuit Breaker Pattern
- Prevent cascading failures by stopping calls to failing downstream services
- **States**: Closed (normal) → Open (failing, reject calls) → Half-Open (test recovery)
- **APIM**: backend circuit breaker configuration (trip on 5xx count/percentage)
- **Logic Apps**: implement with stateful variables or external state (Redis, Table Storage)
- **Service Bus**: natural circuit breaking via dead-letter + MaxDeliveryCount

### Compensation Logic
- When a multi-step transaction partially fails, undo completed steps
- **Saga pattern**: each step has a compensating action (e.g., "cancel reservation" compensates "make reservation")
- Logic Apps `Scope` + `runAfter: Failed` implements compensation blocks
- Durable Functions orchestrator with try/catch for each activity

### Idempotency
- **Event Grid**: at-least-once delivery — consumers MUST be idempotent
- **Service Bus**: PeekLock can redeliver on lock timeout — consumers MUST handle reprocessing
- **Strategies**: check-before-write (DB lookup), deduplication table (message ID), natural idempotency (upsert instead of insert)
- Service Bus supports **duplicate detection** (time window-based, using MessageId) — catches publisher duplicates

## Troubleshooting Integration Flows

### Logic Apps
- **Run history**: shows every execution with input/output per action — primary debugging tool
- **Tracked properties**: add custom properties to action outputs for querying in Log Analytics
- **Diagnostic settings**: send to Log Analytics for cross-workflow analysis and alerting
- **Resubmit**: replay failed runs from the portal (Consumption) or trigger history (Standard)

### Service Bus
- **Service Bus Explorer** (portal): peek, receive, send, dead-letter management
- **Application Insights**: SDK telemetry for send/receive operations
- **Azure Monitor metrics**: active messages, dead-letter count, incoming/outgoing messages, server errors
- **Alerts**: dead-letter count > threshold, active message count growing, server errors

### Event Grid
- **Delivery metrics**: delivery success/failure counts per subscription
- **Dead-letter storage**: inspect undelivered events in blob container
- **Diagnostic logs**: publish/delivery failures with error details

### APIM
- **Request tracing**: `Ocp-Apim-Trace: true` header for detailed policy execution trace
- **Application Insights integration**: full request/response logging, dependency tracking
- **Built-in analytics**: dashboard for usage, response times, failures
- **Diagnostic logs**: gateway logs to Log Analytics
