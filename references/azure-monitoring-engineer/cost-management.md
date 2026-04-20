# Monitoring Cost Management

Monitoring costs can grow quickly if not managed. Understand the cost model and optimize proactively.

## Log Analytics Pricing

| Model | Rate | Best For |
|-------|------|----------|
| Pay-as-you-go | ~$2.76/GB ingestion | Low volume (<100 GB/day) |
| Commitment tiers | 100-5000 GB/day | Predictable volume, up to 30% discount |
| Retention (free) | First 31 days included | Default |
| Retention (paid) | Per-GB beyond 31 days | Up to 730 days interactive |
| Archive tier | Lower per-GB rate | Long-term up to 2,556 days (7 years), limited queries |

## Cost Drivers

The biggest monitoring costs typically come from high-volume log ingestion:

- **NSG flow logs**: Very high volume at full fidelity — can generate GB/hour per NSG
- **AKS container logs**: stdout/stderr from noisy applications
- **API Management diagnostic logs**: Full request/response body logging
- **Application Insights**: High-traffic apps without sampling configured
- **Custom metrics**: High-cardinality custom metrics with many dimensions

## Cost Reduction Strategies

1. **Basic logs tier**: Use for verbose, non-critical data
   - Cheaper ingestion rate
   - Limited query capability (single table, no joins)
   - Good for: container stdout/stderr, verbose debug logs
2. **Data collection rules (DCR)**: Filter and transform before ingestion
   - Drop unnecessary fields or records at collection time
   - Route different log levels to different tables/tiers
3. **Application Insights sampling**: Reduce telemetry volume
   - Adaptive sampling adjusts automatically based on volume
   - Maintains statistical accuracy while reducing cost
4. **Workspace data export**: Archive to storage for long-term retention
   - Cheaper than Log Analytics retention for data older than 31 days
5. **Exclude noisy log categories**: Don't collect what you never query
   - Review ingestion by table — eliminate unused categories

## Common Waste Patterns

- NSG flow logs at full fidelity when traffic analytics summary would suffice
- AKS stdout/stderr from apps that log every request at debug level
- Verbose diagnostic logging left enabled in production after troubleshooting
- Duplicate data sent to multiple workspaces without purpose
- Application Insights with no sampling on high-traffic applications
- AllMetrics enabled in diagnostic settings when Metrics Explorer is sufficient

## Monitoring the Monitor

- **Daily cap**: Set as a safety net on the workspace (not primary cost control — you lose data when hit)
- **Azure Monitor cost analysis workbook**: Built-in template showing ingestion trends by table
- **Ingestion anomaly alerts**: Alert when daily ingestion spikes unexpectedly
- **Regular reviews**: Monthly review of ingestion by table — question any table over 10% of total

## Environment Strategy

| Environment | Diagnostic Level | Sampling | Retention |
|-------------|-----------------|----------|-----------|
| Production | Full categories | Adaptive (moderate) | 90+ days interactive |
| Staging | Reduced categories | Higher sampling rate | 30 days |
| Dev/Test | Minimal diagnostics | Aggressive sampling | 7-30 days |

**Rule of thumb**: If you haven't queried a log category in 6 months, stop collecting it.
