# Cost Management

## Cost Drivers
1. **Data ingestion**: biggest cost — GB/day into Log Analytics workspace
2. **Data retention**: beyond default period (31 days, or 90 days with Sentinel/App Insights)
3. **Log queries**: Basic/Auxiliary tables charge per query
4. **Alerts**: per alert rule evaluation
5. **Prometheus metrics**: per metrics ingested
6. **Application Insights**: per GB telemetry + availability tests

## Cost Optimization Strategies
| Strategy | Impact |
|---|---|
| **Commitment tiers** | 100+ GB/day → tiered pricing discounts (15-25%+) |
| **Basic log tables** | Lower ingestion cost for debug/troubleshooting data |
| **Auxiliary tables** | Lowest cost for compliance/archival data |
| **Data retention config** | Interactive (31-730 days) + long-term archive (up to 12 years) |
| **Summary rules** | Aggregate high-volume data → query summaries instead of raw logs |
| **Daily caps** | Hard stop on ingestion — risky (data loss) but effective budget control |
| **Sampling** | Application Insights adaptive sampling — reduces volume, maintains statistical accuracy |
| **Filter diagnostic settings** | Collect only needed categories — don't enable AllMetrics if already using Metrics Explorer |
| **DCR transformations** | Filter/transform data before ingestion — drop unnecessary fields/records |
| **Workspace Insights** | Monitor ingestion trends, identify anomalies, find top data sources |

## Cost Monitoring KQL
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

## The Observability vs Spend Tradeoff
- **Production**: full telemetry, longer retention, all diagnostic categories
- **Staging**: reduced retention, fewer diagnostic categories, lower sampling rate
- **Dev/Test**: minimal diagnostics, short retention, aggressive sampling
- **Rule of thumb**: if you've never queried a log category in 6 months, stop collecting it
