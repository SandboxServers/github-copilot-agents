# The Three Pillars of Observability

## Metrics
- Numerical values collected at regular intervals, stored in time-series database
- Optimized for fast alerting and real-time detection — preaggregated, low latency
- **Platform metrics**: free, automatic, no configuration, collected from Azure resources
- **Custom metrics**: from applications, agents, REST API — stored per subscription
- **Prometheus metrics**: from AKS/Kubernetes, stored in Azure Monitor workspace, queried with PromQL
- **Best for**: alerting, real-time dashboards, detecting performance trends, threshold monitoring
- **Limitation**: can tell you WHAT is happening but usually not WHY — combine with logs for root cause

## Logs
- Semi-structured data stored in Log Analytics workspace — queried with KQL
- Table plans control cost vs capability:
  - **Analytics**: full KQL, 30-day interactive retention (90 days with Sentinel/AppInsights), up to 12 years total, alert rules, all features
  - **Basic**: lower ingestion cost, limited KQL (single table), no alert rules, per-query charges, good for debug/troubleshooting tables
  - **Auxiliary**: lowest cost, limited KQL, up to 12 years retention, no replication support
- **Best for**: root cause analysis, correlation, complex queries, long-term investigation, compliance

## Distributed Traces
- Track requests across service boundaries — the "call stack" for distributed systems
- W3C Trace Context standard: `traceparent` (trace-id + parent-id) and `tracestate`
- Application Insights maps: `trace-id` → `Operation_Id`, `parent-id` → `Operation_ParentId`
- **Best for**: understanding end-to-end request flow, identifying which service is slow/failing

## When to Use Which
```
Need real-time alerting on thresholds?     → Metrics
Need to investigate root cause?            → Logs (KQL)
Need to trace request across services?     → Distributed traces
Need to detect anomalies automatically?    → Smart detection (App Insights)
Need to understand traffic patterns?       → Metrics + Logs together
Need long-term trend analysis?             → Logs with summary rules
Need Kubernetes monitoring?                → Prometheus metrics + Container Insights
```
