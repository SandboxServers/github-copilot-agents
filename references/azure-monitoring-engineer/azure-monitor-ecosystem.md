# Azure Monitor Ecosystem

Azure Monitor is the unified monitoring platform for all Azure resources, applications, and infrastructure.

## Core Components

- **Metrics**: Numeric time-series data collected at regular intervals
  - Platform metrics collected automatically from Azure resources (free)
  - Custom metrics from applications and agents
  - 93-day retention for platform metrics
  - 1-minute granularity, low-latency access
  - Metrics Explorer for interactive charting and analysis
- **Logs**: Rich structured data stored in Log Analytics workspaces
  - Queried using KQL (Kusto Query Language)
  - Configurable retention from 30 days to 730 days interactive
  - Archive tier extends to 2,556 days (7 years)
  - Basic logs tier available for cost-effective storage of verbose data
- **Application Insights**: Application Performance Management (APM)
  - Auto-instrumentation for .NET, Java, Node.js, Python, JavaScript
  - Workspace-based (backed by Log Analytics workspace)
  - Features: Application Map, Live Metrics, Smart Detection, Availability Tests
  - Transaction diagnostics for end-to-end request tracing
  - Profiler and Snapshot Debugger for .NET
- **Alerts**: Automated notifications and actions based on conditions
  - Metric alerts, log search alerts, activity log alerts, smart detection
  - Action groups for routing notifications and triggering automation
- **Workbooks**: Interactive parameterized reports combining metrics, logs, and text
- **Dashboards**: Portal-based pinnable views for operational overview

## Data Flow

```
Azure Resources → Diagnostic Settings → Log Analytics Workspace + Metrics Store
Applications → App Insights SDK / Auto-instrumentation → Log Analytics Workspace
Kubernetes → Container Insights / Prometheus → Azure Monitor Workspace
VMs → Azure Monitor Agent + Data Collection Rules → Log Analytics Workspace
```

## Log Analytics Workspace Design

- **Centralized (recommended for most orgs)**: Single workspace for all resources
  - Cost-efficient — easier to reach commitment tier discounts
  - Simpler cross-resource queries and correlation
  - Use table-level RBAC for access control
- **Distributed**: Separate workspaces per team, application, or environment
  - Stronger access boundaries
  - Useful when teams have different retention or compliance requirements
  - Cross-workspace queries still possible but more complex
- **Hybrid**: Centralized with Sentinel in separate workspace
  - Security data has different pricing (3-month free retention with Sentinel)
  - Allows security team independent access and retention policies

## Retention and Archive

| Tier | Retention | Cost | Query Capability |
|------|-----------|------|------------------|
| Interactive (default) | 30 days free, up to 730 days | Per-GB beyond free period | Full KQL |
| Archive | Up to 2,556 days (7 years) | Lower storage cost | Limited — restore or search jobs |
| Basic logs | 30 days | Lower ingestion cost | Limited KQL (single table, no joins) |

## Data Collection Rules (DCR)

- Modern replacement for legacy agents and diagnostic settings
- Define what data to collect, how to transform it, and where to send it
- Support filtering and transformation before ingestion (reduce cost)
- Applied via Azure Monitor Agent (AMA) for VMs and scale sets
- Can route different data streams to different tables or workspaces
- **Best practice**: Use DCRs for all new data collection — legacy diagnostic settings still work but DCRs are the future

## Key Integrations

- **Azure Sentinel**: SIEM built on Log Analytics — security monitoring and incident response
- **Azure Managed Grafana**: Industry-standard dashboards with native Azure Monitor data source
- **Event Hubs**: Stream monitoring data to external systems (Splunk, Datadog, etc.)
- **Azure Automation**: Runbooks triggered by alerts for auto-remediation
- **Logic Apps / Azure Functions**: Custom alert response workflows
- **Power BI**: Business-level reporting from Log Analytics data
