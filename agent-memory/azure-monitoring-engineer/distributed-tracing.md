# Distributed Tracing

Distributed tracing tracks requests as they flow through multiple services, providing end-to-end visibility into microservice architectures.

## Application Insights

- Auto-instrumentation available for .NET, Java, Node.js, Python
- Workspace-based Application Insights stores trace data in Log Analytics
- W3C Trace Context standard for cross-service correlation
- `traceparent` header propagated automatically across HTTP boundaries
- `operation_Id` = trace-id, `operation_parentId` = parent span link

## Application Map

- Visual dependency graph showing all connected services
- Displays call rates between services
- Shows error rates per dependency
- Highlights latency between components
- Identifies bottlenecks at a glance
- Click any node to drill into performance and failures

## End-to-End Transaction

- Click on any request (success or failed) to see the full call chain
- Gantt chart visualization showing timing of each operation
- Every dependency call displayed with duration and status code
- Exceptions linked to the specific operation that caused them
- See exactly where time is spent in the request pipeline

## Sampling

Sampling reduces telemetry volume while maintaining statistical accuracy:

- **Adaptive sampling** (default in .NET): Automatically adjusts rate based on volume
- **Fixed-rate sampling**: Configurable percentage (e.g., 25% of all requests)
- **Ingestion sampling**: Applied server-side after data is sent
- **Guidance**: Start with adaptive sampling, adjust if you're missing important data
- **Trade-off**: Lower sampling = lower cost but may miss rare errors

## Custom Telemetry

- `TelemetryClient.TrackEvent()` — business events (order placed, user signed up)
- `TelemetryClient.TrackMetric()` — custom measurements
- `TelemetryClient.TrackDependency()` — manual dependency tracking for unsupported protocols
- Add custom properties for business context: `telemetry.Properties["CustomerId"] = id`
- Use `Activity` class for operation correlation in custom code

## Cross-Service Correlation Patterns

| Transport | Correlation Method |
|-----------|-------------------|
| HTTP | Automatic — `traceparent` header propagated by SDK |
| Service Bus | Set `Diagnostic-Id` on message properties |
| Event Grid | CloudEvents `traceparent` extension attribute |
| Storage Queues | Manual — add correlation ID to message body |
| gRPC | Automatic — propagated via metadata headers |

## What to Instrument

- **Log**: Business events, state transitions, errors, authentication events
- **Measure**: Request duration, dependency duration, queue depth, cache hit rate
- **Trace**: Cross-service requests, async workflows, background job completion
- **Avoid**: PII/secrets, high-frequency debug noise in production, data already captured by auto-instrumentation
- Use structured logging with key-value properties, not string concatenation
