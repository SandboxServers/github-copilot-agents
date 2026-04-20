# Azure Managed Grafana Integration Patterns

Patterns for integrating Grafana with Azure Monitor for visualization and dashboarding.

## Grafana Options on Azure

| Option | Cost | Best For | Limitations |
|---|---|---|---|
| **Dashboards with Grafana (Preview)** | Free (included in Azure Monitor) | Azure-only data sources, quick setup, portal-integrated | No alerts, reports, library panels, snapshots, playlists, or app plugins |
| **Azure Managed Grafana** | Paid (Essential/Standard tiers) | Multi-cloud, external data sources, advanced alerting, enterprise plugins | Requires dedicated instance, separate management |
| **Self-hosted Grafana OSS** | Infrastructure cost only | Full control, custom plugins, air-gapped environments | Operational burden, no SLA, manual upgrades |

## Data Sources for Azure Managed Grafana

| Data Source | Query Language | Use Case |
|---|---|---|
| **Azure Monitor Metrics** | Metric queries | Platform metrics (CPU, memory, DTU, request count) |
| **Azure Monitor Logs** | KQL | Log Analytics queries, custom analysis |
| **Azure Monitor managed Prometheus** | PromQL | Kubernetes cluster metrics, container monitoring |
| **Application Insights Traces** | KQL | Distributed tracing visualization |
| **Azure Resource Graph** | KQL | Resource inventory, compliance dashboards |
| **Azure Data Explorer** | KQL | Large-scale analytics, custom data stores |

## Integration Patterns

### Kubernetes Monitoring Stack

Standard pattern for AKS observability:

1. **Enable Prometheus scraping** → Azure Monitor managed service for Prometheus
2. **Deploy Managed Grafana** → Connect to Azure Monitor workspace
3. **Import community dashboards** → Kubernetes cluster overview, node exporter, pod metrics
4. **Add Container Insights** → stdout/stderr logs, Kubernetes events via Log Analytics
5. **Create custom dashboards** → Application-specific metrics, business KPIs

### Multi-Cloud Monitoring

Grafana as unified pane for Azure + AWS + GCP:

- Azure Monitor data source for Azure resources
- CloudWatch data source plugin for AWS
- BigQuery data source plugin for GCP
- Single dashboard showing cross-cloud SLIs

### Alert Routing with Grafana

Azure Managed Grafana supports its own alerting engine:

- Define alert rules in Grafana using any connected data source
- Route to contact points: email, Slack, PagerDuty, Teams webhook
- Use notification policies for severity-based routing
- **Note**: Dashboards with Grafana (free) does NOT support alerts — use Azure Monitor alerts instead

## Dashboard Design Patterns

| Pattern | Description |
|---|---|
| **Golden Signals dashboard** | Latency, traffic, errors, saturation — one row per service |
| **RED method** | Rate, Errors, Duration — focused on request-driven services |
| **USE method** | Utilization, Saturation, Errors — focused on infrastructure resources |
| **SLO dashboard** | Error budget burn rate, SLI trends, compliance percentages |
| **Deployment correlation** | Overlay deployment markers on metric time series to correlate changes with impact |

## KQL in Grafana

When using Azure Monitor Logs data source in Grafana:

```kql
// Time range is injected by Grafana — use $__timeFilter macro
requests
| where $__timeFilter(timestamp)
| summarize count(), avg(duration) by bin(timestamp, $__interval)
| order by timestamp asc
```

Key macros:
- `$__timeFilter(column)` — maps to Grafana's time range picker
- `$__interval` — auto-calculated bin size based on dashboard time range
- `$__from` / `$__to` — raw timestamps for custom filtering

## Operational Considerations

### Authentication & Access

- Managed Grafana uses Entra ID managed identity for data source access.
- Assign `Monitoring Reader` role on target resources/subscriptions.
- Grafana Admin/Editor/Viewer roles map to Entra ID role assignments on the Grafana workspace.
- Use managed identity — avoid storing credentials in data source configuration.

### Backup & Version Control

- Export dashboards as JSON. Store in Git alongside IaC.
- Automate with Grafana HTTP API or Terraform `grafana_dashboard` resource.
- Tag dashboard versions in Git to enable rollback.

### Performance Tuning

- Complex KQL queries in Grafana can be slow on large datasets.
- Use `materialized_view` or pre-aggregated tables for heavy dashboards.
- Cache timeout defaults to 0 — set appropriate cache duration for stable dashboards.
- Limit time ranges on heavy panels (e.g., max 7 days for detailed log queries).

### Cost Considerations

- Managed Grafana Essential tier: lower cost, single instance, no HA.
- Standard tier: high availability, deterministic IP, private link support.
- Dashboards with Grafana (portal): free, but no alerting or advanced features.
- KQL queries from Grafana count toward Log Analytics query costs.

## References

- [Visualize Azure Monitor data with Grafana](https://learn.microsoft.com/azure/azure-monitor/visualize/visualize-grafana-overview)
- [What is Azure Managed Grafana?](https://learn.microsoft.com/azure/managed-grafana/overview)
- [Azure Managed Grafana FAQ](https://learn.microsoft.com/azure/managed-grafana/faq)
