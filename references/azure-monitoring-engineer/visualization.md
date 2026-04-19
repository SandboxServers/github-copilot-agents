# Visualization

## Workbooks
- Interactive, parameterized reports built into Azure Monitor — no additional cost
- Combine metrics, logs, and text in single document
- Parameters: dropdowns, time range pickers, resource selectors
- Templates: prebuilt for many Azure services
- **Best for**: operational runbooks, capacity planning reports, SLA dashboards

## Azure Dashboards
- Single pane of glass in Azure portal — pin metrics charts, log query results, resource health
- Shareable across team/organization
- **Best for**: quick operational overview, NOC displays

## Grafana (Azure Managed)
- Open-source visualization platform — Azure Managed Grafana as fully managed service
- Native data sources: Azure Monitor, Prometheus, Azure Data Explorer
- **Best for**: Kubernetes monitoring, multi-cloud/hybrid, teams with existing Grafana expertise
- Interoperable with open-source and third-party tools
- Additional cost beyond Azure Monitor

## Power BI
- Business analytics — import log query results for KPI dashboards
- **Best for**: executive/business reporting, long-term trend analysis, sharing outside Azure portal

## Dashboard Design Principles
1. **Answer a question** — every panel should answer a specific operational question. "Is the service healthy?" not "Here are some numbers"
2. **Layer dashboards**: overview → service → component → resource
3. **Use consistent time ranges** — don't mix 1-hour and 24-hour panels without clear labels
4. **Red/yellow/green** — use color to indicate status, not just decoration
5. **Include context** — show related metrics together (latency + throughput + errors)
6. **Use summary rules** for high-volume data — query aggregated data instead of raw logs
