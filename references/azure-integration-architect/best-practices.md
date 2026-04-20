# Best Practices

Proven patterns for Azure integration architecture, messaging, and API management.

## Idempotent Message Handlers

- Every message consumer must be idempotent — at-least-once delivery is the norm across Service Bus, Event Grid, and Event Hubs
- Use the message `MessageId` (Service Bus) or event `id` (Event Grid) for deduplication
- Store processed IDs in a fast lookup: Redis, Azure Table Storage, or a SQL dedup table with TTL cleanup
- Use conditional writes (ETags, `IF NOT EXISTS`) in downstream stores to prevent duplicate side effects
- Service Bus supports duplicate detection natively — enable on queue/topic with a detection window (default 10 minutes, max 7 days)
- Design operations as upserts rather than inserts where possible

## Dead Letter Monitoring and Reprocessing

- Monitor `deadLetterMessageCount` on every queue and topic subscription — alert on any value > 0
- Build an automated DLQ processor: read message, log failure reason, attempt resubmission or route to manual review
- Tag resubmitted messages with a custom property (`resubmitCount`) to prevent infinite loops
- Set a maximum resubmission count — after N retries, archive to blob storage for manual investigation
- DLQ messages include `DeadLetterReason` and `DeadLetterErrorDescription` — log these for root cause analysis
- Schedule DLQ processing on a timer (Azure Function with timer trigger) — don't rely on manual checks

## APIM as Single Entry Point

- Route all external API traffic through APIM — consistent authentication, rate limiting, and observability
- Apply cross-cutting policies at the product or API level: JWT validation, CORS, IP filtering, rate limiting
- Use APIM named values and backends for environment-specific configuration — avoid hardcoded URLs
- Enable Application Insights integration on APIM for request/response logging and dependency tracking
- Use APIM revisions for non-breaking changes, versions for breaking changes
- Keep internal service-to-service calls off APIM unless you need policy enforcement — avoid unnecessary latency

## API Versioning Strategy

- Use URL path versioning (`/v1/orders`) for maximum discoverability and cache-friendliness
- APIM supports URL path, query string, and header-based versioning — choose one strategy and enforce consistency
- Version sets in APIM group related API versions — use them to manage lifecycle (preview, GA, deprecated)
- Set sunset dates on deprecated versions — communicate via response headers (`Sunset`, `Deprecation`)
- Never remove a published API version without at least 6 months deprecation notice

## Circuit Breaker Pattern for External Calls

- Wrap external service calls with circuit breaker logic — prevent cascading failures when a dependency is down
- Use Polly (in .NET), resilience4j (Java), or APIM's built-in `retry` + `send-request` policies
- APIM circuit breaker: configure via backend entity with `circuitBreaker` property — trips after N failures in a time window
- Three states: Closed (normal), Open (fail-fast), Half-Open (probe) — each with configurable thresholds
- Combine with retry policies — retry on transient errors, circuit-break on sustained failures
- Return meaningful fallback responses (cached data, degraded response) rather than raw 503 errors

## Saga Pattern for Distributed Transactions

- Never use distributed transactions (2PC/DTC) across cloud services — they don't scale and most Azure services don't support them
- Implement the saga pattern: a sequence of local transactions with compensating actions for each step
- Orchestration-based: a central coordinator (Logic Apps, Durable Functions) manages the saga sequence
- Choreography-based: each service publishes events, next service reacts — no central coordinator
- Every forward action needs a compensating action: create order → cancel order, charge payment → refund payment
- Store saga state persistently — use Durable Functions entity or a saga state table for recovery after failures

## Event-Driven Architecture: Events vs Commands

- Events describe facts that happened: `OrderPlaced`, `PaymentReceived` — publishers don't know or care about subscribers
- Commands request actions: `ProcessPayment`, `ShipOrder` — sender expects a specific handler to act
- Use Event Grid or Event Hubs for events (fan-out, loose coupling)
- Use Service Bus queues for commands (point-to-point, guaranteed delivery)
- Never put business logic in the event publisher — subscribers own their reactions to events
- Design events as immutable facts — include enough context for subscribers to act without calling back to the publisher

## Retry Policies: Exponential Backoff with Jitter

- Always use exponential backoff — fixed-interval retries amplify load spikes (thundering herd)
- Add jitter (randomized delay) to prevent synchronized retry storms from multiple clients
- Recommended: initial delay 1s, multiplier 2x, max delay 30-60s, max attempts 3-5
- Service Bus SDK has built-in retry — configure `RetryOptions` with `MaxRetries`, `Delay`, `MaxDelay`, `Mode = Exponential`
- APIM: use the `retry` policy with `count`, `interval`, `first-fast-retry`, and condition expressions
- Logic Apps: configure retry policy per action — `exponential` type with `count`, `interval`, `minimumInterval`, `maximumInterval`
- Always set a maximum retry count — infinite retries consume resources and mask permanent failures

## End-to-End Transaction Correlation

- Use Application Insights with distributed tracing across all integration components
- Propagate `traceparent` / `correlation-id` headers through every service call, message, and event
- Service Bus: set `CorrelationId` on messages — Application Insights auto-correlates when using the SDK
- APIM: use `set-header` policy to inject or forward correlation IDs to backends
- Logic Apps: enable tracked properties on actions to include custom data in Application Insights queries
- Build cross-service transaction views using Application Insights Transaction Search and Application Map
- Alert on end-to-end latency percentiles (P95, P99) — not just individual service health

## Claim-Check Pattern for Large Payloads

- Service Bus Standard limits messages to 256 KB — most real-world payloads exceed this quickly
- Store the payload in Azure Blob Storage, send a reference (blob URL + SAS token) in the message
- Consumer retrieves the payload from blob storage using the reference
- Benefits: reduces messaging costs, avoids size limits, enables payload reuse across consumers
- Clean up blobs on a TTL schedule — don't rely on consumers to delete after processing

## Schema Validation at Ingress

- Validate message schemas at the earliest integration boundary — reject invalid data before it propagates
- APIM: use `validate-content` policy to validate request/response bodies against OpenAPI schemas
- Service Bus: no built-in validation — implement in the first consumer or a validation function before enqueueing
- Dead-letter invalid messages immediately with a clear reason — don't attempt partial processing
- Version schemas explicitly — include schema version in message properties for consumer compatibility checks

## Infrastructure as Code for Integration Resources

- Define all queues, topics, subscriptions, Event Grid subscriptions, and APIM configurations in Bicep or Terraform
- Never create integration resources manually in the portal — configuration drift causes environment mismatches
- Use parameter files for environment-specific values (connection strings, tier selections, capacity)
- Include message TTL, lock duration, max delivery count, and duplicate detection settings in IaC — don't rely on defaults
- Deploy integration infrastructure before application code — broken references cause cascading startup failures
