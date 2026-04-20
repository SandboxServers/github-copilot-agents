# KQL Patterns

Essential Kusto Query Language patterns for Azure monitoring and operations.

## Application Insights Queries

```kql
// Find errors in the last 24 hours
AppExceptions
| where TimeGenerated > ago(24h)
| summarize Count=count() by ProblemId, outerMessage
| order by Count desc
| take 20

// Slow requests (p95 latency)
AppRequests
| where TimeGenerated > ago(1h)
| summarize p95=percentile(DurationMs, 95), avg=avg(DurationMs), count=count() by Name
| where p95 > 1000
| order by p95 desc

// Dependency failures
AppDependencies
| where TimeGenerated > ago(1h)
| where Success == false
| summarize FailCount=count(), AvgDuration=avg(DurationMs) by Target, Name, ResultCode
| order by FailCount desc
```

## Azure Resource Queries

```kql
// Resource health changes and failed operations
AzureActivity
| where TimeGenerated > ago(7d)
| where OperationNameValue has "Microsoft.Resources"
| where ActivityStatusValue == "Failed"
| project TimeGenerated, OperationNameValue, Caller, ActivityStatusValue, Properties

// VM CPU over 90%
Perf
| where TimeGenerated > ago(1h)
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize AvgCPU=avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
| where AvgCPU > 90
```

## Kubernetes Queries

```kql
// Container insights - pod restarts
KubePodInventory
| where TimeGenerated > ago(24h)
| where RestartCount > 0
| summarize TotalRestarts=sum(RestartCount) by Namespace, Name
| order by TotalRestarts desc
```

## Error Rate and SLA Queries

```kql
// Error rate as percentage over time
AppRequests
| where TimeGenerated > ago(1h)
| summarize Total=count(), Failures=countif(Success == false) by bin(TimeGenerated, 5m)
| extend ErrorRate = round(100.0 * Failures / Total, 2)
| render timechart

// End-to-end transaction trace
union AppRequests, AppDependencies, AppExceptions, AppTraces
| where OperationId == "specific-operation-id"
| project TimeGenerated, Type, Name, DurationMs, Success, ResultCode
| order by TimeGenerated asc
```

## Cost and Ingestion Queries

```kql
// Data volume by table in last 30 days
Usage
| where TimeGenerated > ago(30d)
| summarize GB = sum(Quantity) / 1000 by DataType
| order by GB desc

// Daily ingestion trend
Usage
| where TimeGenerated > ago(30d)
| summarize DailyGB = sum(Quantity) / 1000 by bin(TimeGenerated, 1d)
| render timechart
```

## KQL Tips

- **Use `ago()` not absolute times** — queries stay relevant without editing
- **`summarize`** for aggregation — count(), avg(), percentile(), min(), max(), dcount()
- **`project`** to select specific columns — reduce noise in results
- **`extend`** to add computed columns — calculations without losing existing columns
- **`let`** for variables — name intermediate results for readability
- **`join`** for cross-table correlation — combine data from different sources
- **`render`** for quick visualization — timechart, barchart, piechart, table
- **Always filter by time first** — `where TimeGenerated > ago(1h)` should be early in the query
- **`summarize` before `render`** — don't chart raw rows
- **Save queries to query packs** — share and reuse across the team
- **Use `series_decompose_anomalies()`** for ML-based anomaly detection in time series
