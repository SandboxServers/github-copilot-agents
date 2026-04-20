# Three Pillars of Observability

The three pillars — metrics, logs, and traces — provide complementary views of system behavior. Each has distinct strengths and all three are needed for full observability.

## Metrics

Numeric time-series data collected at regular intervals. Optimized for speed and alerting.

- **Characteristics**: Low latency, 1-minute granularity, preaggregated, compact storage
- **Platform metrics**: Free, automatic, collected from all Azure resources
- **Custom metrics**: From applications via SDK, agents, or REST API
- **Use for**: Dashboards, autoscaling triggers, fast alerting, trend detection
- **Key examples**:
  - CPU percentage, memory percentage
  - Request rate, error rate
  - Latency percentiles: p50, p95, p99
  - Queue depth, connection count
  - Disk IOPS, network throughput
- **Limitation**: Tells you WHAT is happening, not WHY

## Logs

Structured events with full context. Higher latency but rich detail for investigation.

- **Characteristics**: Semi-structured, variable schema, queried with KQL
- **Types**:
  - **Diagnostic logs**: Resource-specific data plane operations
  - **Application logs**: Custom telemetry from your code
  - **Activity logs**: Control plane operations (who did what to which resource)
- **Use for**: Troubleshooting, auditing, compliance, forensics, root cause analysis
- **Strengths**: Arbitrary queries, cross-resource correlation, long-term retention
- **Query patterns**: Filter → summarize → visualize with KQL

## Traces

Distributed tracing across service boundaries. Shows request flow through multiple services.

- **Characteristics**: Correlation IDs linking operations across services
- **W3C Trace Context**: `traceparent` header propagated across HTTP boundaries
- **Application Map**: Visual dependency graph in Application Insights
- **Transaction diagnostics**: Gantt chart of all operations in a single request
- **Use for**: Identifying bottlenecks, understanding dependencies, root cause analysis in microservices
- **Strengths**: End-to-end visibility, pinpoint which service introduced latency or failure

## How They Complement Each Other

The three pillars work together in a diagnostic workflow:

```
Metrics ALERT → Logs INVESTIGATE → Traces PINPOINT
```

**Example workflow**:
1. **Metric alert fires**: p99 latency spike detected on API service
2. **Query logs**: Find error patterns — increased 500 responses from payment dependency
3. **Trace specific request**: Follow a slow request through API → Payment → Database
4. **Root cause found**: Database connection pool exhaustion causing payment service timeouts

## When to Use Which

| Need | Pillar | Why |
|------|--------|-----|
| Real-time alerting on thresholds | Metrics | Fast, low latency, numeric |
| Root cause investigation | Logs | Rich context, arbitrary queries |
| Cross-service request flow | Traces | Correlation across boundaries |
| Dashboards and trends | Metrics | Compact, fast to render |
| Compliance and auditing | Logs | Detailed records, long retention |
| Bottleneck identification | Traces | Visual timing of each hop |
| Anomaly detection | Metrics + Smart Detection | ML-based baseline comparison |
