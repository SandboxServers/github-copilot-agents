# Azure Monitoring Best Practices

Proven patterns for building effective, cost-efficient observability on Azure.

## Foundation Practices

### Diagnostic settings on every resource from deployment

Include diagnostic settings in ARM/Bicep/Terraform templates for every resource.
Enforce with Azure Policy `DeployIfNotExists`. Every resource should ship logs
to Log Analytics from minute one.

- Route platform metrics and resource logs to a central Log Analytics workspace.
- Route security-relevant logs to Microsoft Sentinel in addition to Log Analytics.
- Audit compliance with Azure Resource Graph queries.

### Centralized Log Analytics workspace with RBAC

Start with one workspace per environment (dev/staging/prod).
Use resource-context RBAC to control access by resource, not by workspace.

- Multiple workspaces add management complexity and cross-query cost.
- Regional data sovereignty may require region-specific workspaces.
- Workspace-per-team is an anti-pattern — use resource-context access instead.

### Structured logging with correlation IDs

Use structured logging (key-value properties, not string interpolation).
Propagate W3C TraceContext correlation IDs across all services.

- Enables cross-service queries: `where customDimensions.CorrelationId == "abc"`
- Use ILogger message templates in .NET: `"Processing {OrderId}"` not `$"Processing {orderId}"`
- Include: CorrelationId, UserId (when permitted), OperationName, Environment

### Application Insights for distributed tracing

Instrument all application services with Application Insights (OpenTelemetry-based).

- Automatic dependency tracking for HTTP, SQL, Service Bus, Redis
- End-to-end transaction view across services
- Application Map visualization of service topology
- Use connection strings (not instrumentation keys — being deprecated)

## Alert Design

### Actionable alerts with runbook links

Every alert rule should include: what broke, who owns it, and how to respond.

- Include runbook URL in alert description
- Use custom properties to enrich payloads: environment, team, service name
- Include escalation path and SLA targets in the runbook
- Assign every alert to an owner — unowned alerts become noise

### Dynamic thresholds for unknown baselines

Use Azure Monitor dynamic thresholds when you don't have established baselines.

- ML-based detection learns patterns (daily cycles, weekly trends)
- Adjusts automatically — no manual threshold tuning
- Review and tune sensitivity after 2 weeks of learning
- Good for: CPU, memory, request rate, latency

### Alert processing rules for maintenance windows

Suppress alerts during planned maintenance using alert processing rules.

- Scope by resource group, resource type, tag, or specific resources
- Schedule recurring rules for regular maintenance windows
- Avoids alert storms during deployments, patching, and planned failovers
- Can also add action groups to existing alerts without modifying the rules

### Severity classification with ownership

- **Sev 0/1**: Auto-page on-call via PagerDuty/ServiceNow. Production impact.
- **Sev 2**: Team channel notification (Slack/Teams). Service degradation.
- **Sev 3/4**: Email digest or dashboard only. Informational.
- Free alert types: Activity log, Service Health, Resource Health — use them.

## Cost Management

### Basic logs for verbose data

Route verbose trace/debug logs to the Basic logs tier.

- Cheaper ingestion than Analytics tier
- 8-day interactive retention, then archive
- Pay-per-query model — charges only when you read the data
- Trade-off: no alerts, limited KQL, no joins
- Good for: raw request bodies, debug traces, verbose infrastructure logs

### Commitment tiers for predictable workloads

If ingesting 100+ GB/day consistently, commitment tiers save 15-30%.

- Review monthly and adjust tier up or down
- Overage beyond commitment is billed at pay-as-you-go rate
- Start with 100 GB/day tier and scale as needed

### Daily cap as safety net

- Set at 2-3x expected daily volume
- Alert at 80% of cap for advance warning
- Never use as primary cost control — it drops ALL data when hit
- Consider: runaway logging from a bug can consume a month's budget in hours

### Regular ingestion review

Query the `Usage` table monthly to identify top data sources by volume.

```kql
Usage
| where TimeGenerated > ago(30d)
| summarize TotalGB = sum(Quantity) / 1000 by DataType
| sort by TotalGB desc
| take 10
```

Look for noisy tables, unexpected growth, or data nobody queries.
Adjust collection rules, sampling, or table plans accordingly.

## Visualization & Analysis

### Workbooks for interactive dashboards

Use Azure Monitor Workbooks for rich, parameterized, interactive dashboards.

- Combine metrics, logs, and Azure Resource Graph in a single view
- Parameters for time range, subscription, resource group — user-selectable
- Share via Azure portal with RBAC
- Version control workbook JSON templates in Git

### KQL saved queries and functions

Save common queries as functions in Log Analytics for reuse.

- Reuse across workbooks, alert rules, and ad-hoc analysis
- Examples: error rate by service, P95 latency trends, top slow requests
- Version control query logic alongside application code
- Use `let` statements and functions to build composable query libraries

### Availability tests for all public endpoints

- Configure URL ping tests from 5+ Azure regions
- Alert on 2+ consecutive failures to avoid false positives
- Use Standard tests with content validation — check response body, not just HTTP 200
- Custom `TrackAvailability()` for complex user journeys

### Alert processing rules for maintenance windows

Suppress notifications during planned changes without disabling alert rules.
Scope by resource group, tags, or individual resources.

## Data Collection

### Data Collection Rules (DCRs) per scope

Create DCRs specific to observability scope — don't bundle unrelated collection.

- OS basics (CPU, memory, disk): one DCR for all servers
- Application logs: separate DCR per application or tier
- Security events: dedicated DCR routed to Sentinel
- Easier maintenance: update one scope without affecting others

### Azure Monitor Agent (AMA) for all VMs

Migrate from legacy Log Analytics agent (MMA) to AMA.

- AMA supports DCRs, multi-homing, and granular configuration
- Legacy MMA agent is deprecated — plan migration now
- Test DCR configurations in staging before production rollout
- Check data source support matrix — some sources need workarounds

### Sampling strategy for Application Insights

- Use adaptive sampling for general telemetry (auto-adjusts to volume)
- Override to 100% for errors, exceptions, and critical operations
- Account for `itemCount` in all KQL aggregations
- Metrics are never sampled — use them for alerting

## References

- [Azure Monitor best practices](https://learn.microsoft.com/azure/azure-monitor/fundamentals/best-practices-operation)
- [Best practices for Azure Monitor alerts](https://learn.microsoft.com/azure/azure-monitor/alerts/best-practices-alerts)
- [Data collection rule best practices](https://learn.microsoft.com/azure/azure-monitor/data-collection/data-collection-rule-best-practices)
- [Monitoring and alerting strategy — Well-Architected Framework](https://learn.microsoft.com/azure/well-architected/reliability/monitoring-alerting-strategy)
