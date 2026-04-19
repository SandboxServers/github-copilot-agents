# Distributed Tracing

## How It Works
- Each incoming request gets an `operation_Id` (trace-id) — all downstream telemetry shares this ID
- Each operation within the trace gets a unique `id` and links to parent via `operation_parentId`
- Request telemetry → Dependency telemetry → next service's Request telemetry (linked by IDs)
- W3C Trace Context: `traceparent` header propagated across HTTP boundaries

## Application Insights Correlation
- `operation_Id`: root trace identifier — shared by ALL telemetry in the distributed transaction
- `request.id`: unique ID for this specific request
- `dependency.id`: unique ID for outgoing call
- `operation_parentId`: links child to parent span
- `request.source` / `dependency.target`: identify calling/called components

## Key Views
- **Application Map**: visual topology — shows dependencies, call counts, failure rates, avg duration
- **Transaction Diagnostics**: Gantt chart of all operations in a single transaction — find the slow dependency
- **Failures view**: drill from failed requests → failed dependencies → exceptions
- **Performance view**: drill from slow requests → slow dependencies → identify bottlenecks

## Instrumentation Guidance
- **What to log**: business events, state transitions, external calls, errors, authentication events
- **What to measure**: request duration, dependency duration, queue length, cache hit rate, error rate
- **What to trace**: cross-service requests, async workflows, background job completion
- **What NOT to log**: PII/secrets, high-frequency debug noise in production, redundant data already captured by auto-instrumentation
- Use structured logging — key-value properties, not string concatenation
- Set correlation IDs at the entry point — propagate through all downstream calls
