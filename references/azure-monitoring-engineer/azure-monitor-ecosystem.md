# Azure Monitor Ecosystem

## Log Analytics Workspace
- Central store for all log data — resource logs, activity logs, custom logs, Application Insights
- KQL (Kusto Query Language) for querying — same engine as Azure Data Explorer
- Simple mode (point-and-click) and KQL mode (full query language)
- Scope: can query single resource, resource group, subscription, or cross-workspace
- **Workspace design considerations**:
  - Single vs multiple workspaces: balance visibility vs access control vs cost
  - Separate Sentinel workspace: security data has different pricing (3-month free retention)
  - Combined workspace: better cross-correlation, may reach commitment tier discount
  - Region matters: deploy App Insights in same region as workspace to minimize latency/cost
  - Table-level RBAC for fine-grained access control
  - Dedicated cluster: 500+ GB/day for cost discount and additional features

## Application Insights
- APM for web applications — performance monitoring, failure detection, dependency tracking
- Workspace-based (backed by Log Analytics workspace)
- Auto-instrumentation for .NET, .NET Core, Java, Node.js, JavaScript, Python
- Key features:
  - **Application Map**: visual topology of dependencies and their health
  - **Transaction Diagnostics**: end-to-end transaction view with all correlated telemetry
  - **Failures & Performance views**: drill into slow/failed requests and their dependencies
  - **Smart Detection**: ML-based anomaly detection for performance degradation and failure spikes
  - **Availability Tests**: synthetic monitoring from multiple locations
  - **Live Metrics**: real-time telemetry stream
  - **Profiler**: code-level performance profiling (.NET)
  - **Snapshot Debugger**: production debugging on exceptions
- Release annotations: mark deployments on timeline for correlation

## Azure Monitor Metrics
- Time-series database for platform metrics (free) and custom metrics
- Metrics Explorer: interactive analysis, charting, filtering by dimensions
- **Drill into Logs**: correlate metric anomalies with activity logs, diagnostic logs, recommended queries
- Preaggregated at 60-second intervals — custom metrics sent more frequently are buffered
- **Gotcha**: high-cardinality dimensions (>100 unique values per dimension) hit 50,000 time series limit
- **Gotcha**: never use timestamps or unbounded IDs as metric dimensions

## Prometheus Metrics
- Stored in Azure Monitor workspace (separate from Log Analytics workspace)
- Queried with PromQL, visualized with Grafana
- Collected from AKS or any Kubernetes cluster via remote-write
- **Best for**: Kubernetes-native monitoring, when team already uses Prometheus/Grafana
