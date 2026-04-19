# Alerting

## Alert Types
| Type | Signal | Use Case |
|---|---|---|
| **Metric alerts** | Platform/custom metrics | Threshold-based, real-time, dynamic thresholds |
| **Log search alerts** | KQL query results | Complex conditions, multi-resource correlation |
| **Activity log alerts** | Azure control plane events | Resource changes, service health, resource health |
| **Smart detection** | App Insights ML | Automatic anomaly detection — performance degradation, failure spikes |
| **Prometheus alerts** | PromQL queries | Kubernetes workload alerting |

## Action Groups
- Reusable notification/action targets shared across alert rules
- Notifications: email, SMS, push, voice call
- Actions: Azure Functions, Logic Apps, webhooks, ITSM connectors, Automation runbooks, Event Hubs
- **Best practice**: define action groups per severity/team, reuse across rules

## Alert Processing Rules
- Modify triggered alerts at any ARM scope — apply/suppress action groups, filter, schedule
- Use for: maintenance windows (suppress), routing (add action groups), filtering (specific resource types)

## Alerting Best Practices
1. **Alert on symptoms, not causes** — alert on "response time > 2s" not "CPU > 90%". High CPU with fast response time is not a problem.
2. **Alert on rates, not absolutes** — alert on "error rate > 5%" not "5 errors occurred". 5 errors in 1M requests is normal; 5 errors in 10 requests is a crisis.
3. **Alert on what's actionable** — every alert should have a runbook or at least a clear next step. If nobody can act on it, it's noise.
4. **Use dynamic thresholds** — ML-based baselines that adapt to weekly/daily patterns. Avoids false positives from expected traffic variations.
5. **Severity levels matter**:
   - Sev 0 (Critical): page someone NOW — production down
   - Sev 1 (Error): investigate urgently — degraded service
   - Sev 2 (Warning): look at it during business hours — potential issue
   - Sev 3 (Informational): log for awareness — operational insight
   - Sev 4 (Verbose): diagnostic detail
6. **Don't alert on everything** — alert fatigue is worse than no alerting. If people stop looking at alerts, you have no alerting.
7. **Priority gaps**: leave room between threshold values for tighter alerts later
8. **Test alerts** — intentionally trigger alerts during onboarding to verify the pipeline works
