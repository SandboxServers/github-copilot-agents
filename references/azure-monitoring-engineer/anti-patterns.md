# Azure Monitoring Anti-Patterns

Common monitoring mistakes that undermine observability and waste resources.
Source: [Azure Well-Architected Framework — Monitoring Antipatterns](https://learn.microsoft.com/azure/well-architected/design-guides/monitoring#monitoring-antipatterns-and-how-to-avoid-them)

## Alert Fatigue

Too many alerts fire, responders ignore all of them, and signal is lost in noise.

- **Root cause**: Every metric gets an alert "just in case." No severity classification. No ownership.
- **Symptoms**: Hundreds of open alerts. On-call ignores pages. Customers report outages first.
- **Fix**: Classify by severity (Sev 0-4). Assign an owner to every alert.
  Prune quarterly — if nobody acts on it in 90 days, delete or downgrade it.
  Use alert processing rules to suppress during maintenance.

## Monitoring as Afterthought

Observability bolted on after production incidents reveal blind spots.

- **Root cause**: "We'll add monitoring later." Diagnostic settings, Application Insights,
  and alerts not included in IaC templates.
- **Symptoms**: Resources in production with no diagnostic settings.
  First alert rule created during an outage.
- **Fix**: Include diagnostic settings, App Insights instrumentation, and alert rules
  in ARM/Bicep/Terraform from day one. Enforce with Azure Policy `DeployIfNotExists`.
  Monitoring ships with the resource, not after.

## No Correlation Across Signals

Metrics, logs, and traces collected in isolation — can't connect a spike in errors to a specific trace.

- **Root cause**: Each team instruments differently. No shared correlation ID.
  Separate tools for metrics vs logs vs traces.
- **Symptoms**: "We see 500 errors in metrics but can't find the failing request in logs."
- **Fix**: Propagate W3C TraceContext correlation IDs across all services.
  Use Application Insights with a shared connection string.
  Ensure all components emit to the same Log Analytics workspace.

## Sampling Too Aggressively

Fixed-rate sampling at 5% drops 95% of transactions — errors and edge cases vanish.

- **Root cause**: Sampling configured to minimize cost without considering diagnostic value.
- **Symptoms**: "We know there were failures but can't find them in App Insights."
  Queries return `itemCount > 1` on everything.
- **Fix**: Use adaptive sampling (auto-adjusts to volume).
  Configure sampling overrides to keep errors at 100% and slow requests at 100%.
  Always use `summarize sum(itemCount)` instead of `count()` in KQL.

## Not Using Structured Logging

String concatenation instead of structured properties — can't query by individual fields.

- **Bad**: `logger.LogInformation($"Order {orderId} processed in {elapsed}ms")`
  This produces an opaque string. You can only search with `contains`.
- **Good**: `logger.LogInformation("Order {OrderId} processed in {ElapsedMs}ms", orderId, elapsed)`
  Structured properties become queryable columns in Log Analytics.
  You can filter: `where customDimensions.OrderId == "12345"`
- **Why it matters**: Structured logging enables aggregation, filtering, and alerting on
  individual fields without regex parsing.

## Dashboard Overload

50+ metrics on a single dashboard — nobody can find the signal.

- **Root cause**: "Add everything so we don't miss anything." No defined audience.
- **Symptoms**: Dashboard takes 30+ seconds to load. Charts are tiny. Nobody looks at it.
- **Fix**: Design dashboards for decisions, not decoration.
  One dashboard per audience (exec, ops, dev).
  5-10 key metrics max per view. Use drill-down links for details.
  Align to ALES model: Availability, Latency, Errors, Saturation.

## Alerting on Symptoms Instead of Causes

Alert fires for "high CPU" when the real problem is a runaway query or memory leak.

- **Root cause**: Alerting on infrastructure metrics without understanding what drives them.
- **Symptoms**: CPU alert fires → team investigates → finds an app-layer bug.
  Alert didn't help identify root cause.
- **Fix**: Alert on business-impacting conditions: error rate breaches SLO,
  P95 latency exceeds threshold. Use infrastructure metrics (CPU, memory)
  as diagnostic signals in workbooks, not as primary alert triggers.

## No Baseline Established

Thresholds set to arbitrary values ("CPU > 80%") without knowing what "normal" looks like.

- **Root cause**: Alerts created at deployment time with guessed thresholds. Never revisited.
- **Symptoms**: False positives during normal peak hours.
  Thresholds too high to catch real degradation during off-peak.
- **Fix**: Use Azure Monitor dynamic thresholds — ML-based detection that learns patterns.
  For static thresholds, baseline for 2+ weeks before setting values.
  Review thresholds quarterly as workloads evolve.

## Ignoring Custom Metrics

Relying only on platform metrics (CPU, memory, DTU) — no visibility into business logic.

- **Root cause**: Team assumes platform metrics are sufficient.
  No investment in application-level instrumentation.
- **Symptoms**: "Infrastructure looks fine but customers can't check out."
  No metric for orders/sec or payment failures.
- **Fix**: Emit custom metrics for business KPIs: orders/sec, payment success rate,
  queue processing time. Use `TelemetryClient.TrackMetric()`,
  OpenTelemetry meters, or Prometheus custom metrics.

## Log Retention Mismanagement

Retention too short (lose investigation data) or too long at the wrong tier (burn budget).

- **Too short**: 30-day default loses data needed for trend analysis and post-mortems.
  Can't answer "was this happening last month too?"
- **Too long at wrong tier**: 2 years of verbose debug logs at full Analytics pricing
  is extremely wasteful when nobody queries it.
- **Fix**: 90-day interactive retention for operational data.
  Basic logs tier for verbose/debug data (cheaper ingestion, 8-day interactive).
  Archive tier for compliance (up to 12 years, cheap storage, query-time cost).

## Missing Diagnostic Settings

Resources deployed without diagnostic settings — no logs reaching Log Analytics.

- **Root cause**: Diagnostic settings are a separate Azure resource — easy to forget.
- **Symptoms**: Incident investigation reveals zero log data for the affected resource.
  Hours of diagnostic data permanently lost.
- **Fix**: Enforce via Azure Policy (`DeployIfNotExists`).
  Include diagnostic settings in every resource module in your IaC.
  Audit compliance with Azure Resource Graph queries.
  Remember: diagnostic settings don't backfill — every hour without them is data lost.

## Missing Availability Tests

Public-facing endpoints go down and the team finds out from customer complaints.

- **Root cause**: "We'll set up synthetic monitoring later." Never happens.
- **Symptoms**: Customer tweets about downtime before the team knows.
- **Fix**: Configure Application Insights availability tests (URL ping or Standard test).
  Test from 5+ Azure regions. Alert on 2+ consecutive failures to avoid
  false positives from transient network issues.
  Validate response content, not just HTTP 200.

## References

- [Monitoring antipatterns — Azure Well-Architected Framework](https://learn.microsoft.com/azure/well-architected/design-guides/monitoring#monitoring-antipatterns-and-how-to-avoid-them)
- [Best practices for Azure Monitor alerts](https://learn.microsoft.com/azure/azure-monitor/alerts/best-practices-alerts)
- [Sampling in Application Insights](https://learn.microsoft.com/azure/azure-monitor/app/sampling-classic-api)
- [Architecture strategies for monitoring](https://learn.microsoft.com/azure/well-architected/operational-excellence/observability)
