# KQL (Kusto Query Language)

## Essential Patterns
```kql
// Find errors in the last hour
requests
| where timestamp > ago(1h)
| where success == false
| summarize count() by resultCode, bin(timestamp, 5m)
| render timechart

// Slow dependencies
dependencies
| where timestamp > ago(1h)
| where duration > 1000  // > 1 second
| summarize avg(duration), count() by target, name
| order by avg_duration desc

// Error rate as percentage
requests
| where timestamp > ago(1h)
| summarize total = count(), failures = countif(success == false) by bin(timestamp, 5m)
| extend error_rate = round(100.0 * failures / total, 2)
| render timechart

// End-to-end transaction trace
(requests | union dependencies | union exceptions | union traces)
| where operation_Id == "specific-operation-id"
| project timestamp, itemType, name, duration, success, resultCode
| order by timestamp asc

// Top exceptions by type
exceptions
| where timestamp > ago(24h)
| summarize count() by type, outerMessage
| order by count_ desc
| take 20

// Resource log analysis
AzureDiagnostics
| where TimeGenerated > ago(1h)
| where Category == "SQLSecurityAuditEvents"
| summarize count() by event_class_s, bin(TimeGenerated, 5m)
```

## KQL Design Principles
- **Start narrow, expand as needed** — always filter by time range first (`where timestamp > ago(1h)`)
- **Summarize before rendering** — don't render raw rows on a chart
- **Use `let` for readability** — name intermediate results
- **Use `render` for quick visualization** — timechart, barchart, piechart
- **Use dynamic thresholds via `series_decompose_anomalies()`** for anomaly detection
- **Save queries to query packs** for reuse across team
