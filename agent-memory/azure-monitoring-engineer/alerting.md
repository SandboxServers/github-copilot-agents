# Alerting

Alerting is the bridge between monitoring data and human/automated response.

## Alert Types

| Type | Signal Source | Speed | Use Case |
|------|--------------|-------|----------|
| **Metric alerts** | Platform/custom metrics | Fast (1-min eval) | Numeric thresholds, autoscale triggers |
| **Log search alerts** | KQL query results | Slower (5-15 min) | Complex conditions, multi-resource correlation |
| **Activity log alerts** | Azure control plane | Near real-time | Resource changes, deletions, policy violations |
| **Service Health alerts** | Azure service status | As available | Outages, maintenance, advisories |
| **Resource Health alerts** | Resource availability | As available | VM rebooted, SQL failover, degraded state |

## Signal Types

- **Metric**: CPU, memory, requests, error count, latency — numeric values over time
- **Log**: KQL query result — flexible, can combine multiple data sources
- **Activity log**: Resource operations — create, delete, update, role assignments
- **Resource Health**: Availability state changes — available, degraded, unavailable

## Action Groups

Reusable notification and action targets shared across alert rules:

- **Notifications**: Email, SMS, push notification, voice call
- **Automation**: Azure Function, Logic App, Automation Runbook
- **Integration**: Webhook, ITSM connector (ServiceNow, etc.), Event Hub
- **Best practice**: Define action groups per severity and team, reuse across rules

## Smart Detection (Application Insights)

Automatic ML-based anomaly detection — no configuration required:

- Failure rate anomalies (sudden increase in failed requests)
- Performance degradation (response time regression)
- Memory leak detection
- Dependency slowdowns and failures
- Trace severity ratio changes

## Best Practices

1. **Alert on symptoms, not causes** — alert on error rate increasing, not CPU being high. High CPU with fast responses is fine.
2. **Use dynamic thresholds** — ML-based baselines that adapt to weekly/daily patterns. Avoids false positives from expected traffic variations.
3. **Severity levels**:
   - Sev 0 (Critical): Page someone NOW — production down, data loss imminent
   - Sev 1 (Error): Notify urgently — degraded service affecting users
   - Sev 2 (Warning): Address during business hours — potential issue developing
4. **Suppress noise**: Use aggregation windows, auto-resolve periods, and minimum evaluation count
5. **Alert on absence of data** — heartbeat monitoring. If a service stops reporting, that's an alert.
6. **Every alert needs an action** — if nobody knows what to do when it fires, it's noise

## Common Alert Rules

| What to Alert On | Type | Threshold Example |
|-----------------|------|-------------------|
| Error rate spike | Metric / Log | > 5% of requests failing |
| p95 latency breach | Metric | > SLA target (e.g., 2 seconds) |
| Availability drop | Metric | < 99.9% over 5 minutes |
| Dead letter queue depth | Metric | > 0 messages |
| Certificate expiry | Log | < 30 days to expiration |
| Disk space low | Metric | < 20% free |
| No heartbeat | Log | Missing data for > 5 minutes |
| Resource deletion | Activity log | Delete operation on production resources |
