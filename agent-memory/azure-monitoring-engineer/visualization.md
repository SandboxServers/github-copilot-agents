# Visualization

Choose the right visualization tool based on your audience and use case.

## Azure Dashboards

- Pin metrics charts, log query results, markdown tiles to a shared portal view
- Share across team via RBAC
- Limited interactivity — static views with time range selection
- **Best for**: Quick operational overview, NOC wall displays

## Workbooks

- Interactive, parameterized reports built into Azure Monitor
- Combine KQL queries, metrics, and markdown in a single document
- Parameters: dropdowns, time range pickers, resource selectors
- Drill-down navigation between sections
- Templates available for common scenarios (VM performance, container health, etc.)
- No additional cost beyond Azure Monitor
- **Best for**: Operations teams — runbooks, capacity planning, SLA reporting

## Grafana (Azure Managed)

- Industry-standard dashboard platform, fully managed as Azure service
- Native Azure Monitor data source — metrics, logs, and Prometheus
- Community dashboard templates for hundreds of scenarios
- Supports multi-cloud and hybrid monitoring in one place
- Teams already familiar with Grafana can reuse existing skills
- **Best for**: Kubernetes monitoring, teams with Grafana expertise, multi-cloud environments

## Power BI

- Business-level reporting and analytics platform
- Connect to Log Analytics via Azure Monitor Logs connector
- Scheduled refresh for near-real-time dashboards
- Rich formatting and sharing outside the Azure portal
- **Best for**: Executive dashboards, business stakeholders, KPI reporting

## Application Insights Dashboards

- Built-in views that auto-configure with Application Insights:
  - **Application Map**: Visual dependency topology
  - **Live Metrics**: Real-time telemetry stream
  - **Performance**: Request duration analysis with drill-down
  - **Failures**: Error analysis with exception details
  - **Users**: Usage analytics and session flows
- **Best for**: Application developers and SREs investigating specific app behavior

## Best Practices

1. **Layer your visualization** — match the tool to the audience:
   - Executive dashboard (Power BI) → high-level KPIs and trends
   - Operations dashboard (Workbooks/Grafana) → service health and metrics
   - Investigation (Log Analytics KQL) → ad-hoc queries and root cause
   - Deep dive (Application Map, traces) → specific request analysis
2. **One source of truth** — don't duplicate the same metric across multiple dashboards
3. **Every panel answers a question** — "Is the service healthy?" not "Here are some numbers"
4. **Use color for status** — red/yellow/green to indicate health, not decoration
5. **Consistent time ranges** — don't mix 1-hour and 24-hour panels without clear labels
6. **Show related metrics together** — latency + throughput + errors on the same view
