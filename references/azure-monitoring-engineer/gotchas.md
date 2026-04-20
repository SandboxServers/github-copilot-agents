# Azure Monitoring Gotchas

Surprising behaviors, hidden limits, and edge cases that bite even experienced engineers.

## Log Analytics Gotchas

### Ingestion delay up to 5+ minutes

Resource logs typically take 3-10 minutes end-to-end depending on the service.
Activity logs can take 3-20 minutes. After data reaches the collection endpoint,
Azure Monitor processing adds 5-15 seconds.

- Don't rely on real-time log queries for immediate incident detection.
- Use metric alerts for fast detection (1-min evaluation), logs for investigation.
- `_TimeReceived` shows when Log Analytics got the data; `TimeGenerated` is source time.
- Late-arriving data can have `TimeGenerated` hours in the past — be aware in alert evaluations.

### Cross-workspace queries are slower and more expensive

`workspace('other').Table` queries scan remote workspaces over the network.
Performance degrades with data volume and number of workspaces.

- Minimize cross-workspace joins. Consolidate into one workspace where possible.
- Use resource-context RBAC to control access by resource, not workspace-per-team.
- If you must cross-query, add tight time filters and limit result sets.

### Basic logs tier limitations

Basic logs: cheaper ingestion, but comes with significant restrictions.

- 8-day interactive retention only (then archive)
- Limited KQL: no joins, no aggregations across tables
- Cannot create log search alert rules against Basic tier tables
- Query-time charges per GB scanned (pay when you read)
- Use only for verbose data you rarely query: trace logs, debug output, raw telemetry

### Workspace daily cap stops ALL ingestion

When the daily cap is hit, all data stops flowing until the reset time —
including security logs and diagnostic data.

- Use daily cap as a safety net, never as primary cost management.
- Set at 2-3x expected daily volume.
- Create an alert at 80% of the cap to get advance warning.
- Consider commitment tiers instead for cost management.

## Application Insights Gotchas

### Sampling can drop important transactions

Adaptive sampling reduces volume under load — it may discard error traces,
slow requests, or rare but critical operations.

- Configure sampling overrides to keep errors and exceptions at 100%.
- Use fixed-rate sampling with known percentages for critical paths.
- Always use `summarize sum(itemCount)` in KQL — `count()` undercounts sampled data.
- Performance metrics and custom metrics are never sampled.

### Live Metrics ignores sampling

Live Metrics stream shows ALL incoming telemetry regardless of sampling settings.
This gives a misleading view of "true" volume compared to queried data.

- Don't compare Live Metrics counts with query results (queries reflect sampling).
- Live Metrics is useful for real-time debugging but not for volume estimation.

### Connection string vs instrumentation key

Instrumentation keys are being deprecated. Connection strings include the ingestion
endpoint and support regional routing, private link, and sovereign clouds.

- Always use connection strings for new instrumentation.
- Update legacy apps still using `InstrumentationKey=`.
- Connection string format: `InstrumentationKey=...;IngestionEndpoint=...;LiveEndpoint=...`

### Sampling rate hidden in queries

Query results look normal but may represent only a fraction of actual traffic.
The `itemCount` field on each record shows the extrapolation factor.

- Use `summarize sum(itemCount)` instead of `count()` for true volume calculations.
- Check sampling status: query `union requests,dependencies | summarize 100/avg(itemCount)`

## KQL Gotchas

### `contains` is case-sensitive

`contains` performs a case-sensitive substring match.
`contains_cs` is redundant — it does the same thing.
There is no built-in case-insensitive `contains`.

- Use `has` for case-insensitive whole-word matching (also much faster — uses term index).
- Use `tolower(column) contains "value"` if you truly need case-insensitive substring search.
- `has_cs` is the case-sensitive version of `has`.

### `has` vs `contains` performance

`has` uses the inverted term index — orders of magnitude faster on large datasets.
`contains` does a full-text character scan.

- Default to `has` for filtering. Use `contains` only for true substring matching
  (e.g., partial GUIDs, fragments within compound strings).
- Prefer `startswith` or `endswith` over `contains` when the position is known.

### `join` without time filter on inner query

Inner query scans the full retention range of the table even if the outer query is
filtered — causes massive performance degradation and cost.

- Always add explicit `where TimeGenerated > ago(...)` to both sides of a join.
- Use `let MinTime = ago(1d)` for consistent time scoping across subqueries.

### `union` time scope trap

Filtering after `union` forces full table scans, then filters.

- Apply time filter inside each `union` subquery, not after the union.
- Use `let` statements for consistent scoping.

## Alert System Gotchas

### Action group rate limits

Email notifications capped at 100 emails per hour per action group.
SMS: 1 per 5 minutes. Voice call: 1 per 5 minutes.

- Use email for non-critical alerts only.
- Use PagerDuty, ServiceNow, or webhook for high-frequency critical alerts.
- Group related alerts with alert processing rules to reduce notification volume.

### Evaluation frequency vs aggregation mismatch

1-minute evaluation frequency with 5-minute aggregation window creates overlapping
windows — the same data point triggers multiple evaluations, causing duplicate alerts.

- Match evaluation frequency to aggregation window, or ensure frequency ≤ window.
- For metric alerts: if aggregation is PT5M, set evaluation to PT5M.
- For log alerts: set evaluation period ≥ lookback period.

### Stateful alerts auto-resolve behavior

Metric alerts auto-resolve when the condition clears. Log alerts can be configured
for auto-resolve but it's not always the default.

- Configure auto-resolve for log search alerts explicitly.
- Metric alerts: `autoMitigate: true` is default — alerts resolve when condition clears.
- Unresolved log alerts accumulate and create noise in the alerts list.

## Azure Monitor Agent Gotchas

### Not all legacy agent data sources supported

AMA may not support all data sources from the legacy Log Analytics agent (MMA).
Some custom logs, IIS logs, and solutions require specific DCR configurations.

- Check the AMA data source support matrix before migration.
- Plan migration per data source, not per machine.
- Test DCR configurations in a staging environment before production rollout.

### Diagnostic settings category names vary by resource type

Each Azure resource type has its own set of log category names.
Categories are named inconsistently across services (e.g., `AuditEvent` vs `Audit`).

- Don't assume category names are uniform across resource types.
- Check `az monitor diagnostic-settings categories list` for each resource type.
- Use Azure Policy with per-resource-type definitions for enforcement.

### Diagnostic settings don't retroactively collect

Enabling diagnostic settings only captures data going forward. No backfill.

- Enable at resource creation via IaC. Every hour without settings = data permanently lost.
- Use Azure Policy `DeployIfNotExists` to catch resources created outside IaC.

## References

- [Log data ingestion time in Azure Monitor](https://learn.microsoft.com/azure/azure-monitor/logs/data-ingestion-time)
- [Sampling in Application Insights](https://learn.microsoft.com/azure/azure-monitor/app/sampling-classic-api)
- [Optimize log queries in Azure Monitor](https://learn.microsoft.com/azure/azure-monitor/logs/query-optimization)
- [Azure Monitor service limits](https://learn.microsoft.com/azure/azure-monitor/service-limits)
