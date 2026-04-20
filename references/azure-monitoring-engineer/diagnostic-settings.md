# Diagnostic Settings

Every Azure resource in production should have diagnostic settings configured. No exceptions.

## What They Do

Diagnostic settings route platform logs and metrics from Azure resources to one or more destinations for analysis, alerting, and archival.

## Configuration

- **Source**: Any Azure resource that emits platform logs or metrics
- **Destination options**:
  - Log Analytics workspace — for querying with KQL and alerting
  - Storage account — for archival and compliance retention
  - Event Hub — for streaming to external SIEM (Sentinel, Splunk, Datadog)
  - Partner solutions — direct integration with third-party monitoring
- Must be configured per resource — they are NOT automatic
- Multiple diagnostic settings per resource are supported (different destinations)

## Log Categories

- **categoryGroup: 'allLogs'**: Captures all log categories for the resource (recommended)
- **categoryGroup: 'audit'**: Captures audit-related logs only
- **Individual categories**: Fine-grained control per resource type
- **AllMetrics**: Send platform metrics to Log Analytics for KQL analysis

## Bicep Template

```bicep
resource diagnostics 'Microsoft.Insights/diagnosticSettings@2021-05-01-preview' = {
  name: 'send-to-law'
  scope: targetResource
  properties: {
    workspaceId: logAnalyticsWorkspaceId
    logs: [
      {
        categoryGroup: 'allLogs'
        enabled: true
      }
    ]
    metrics: [
      {
        category: 'AllMetrics'
        enabled: true
      }
    ]
  }
}
```

## Azure Policy for Enforcement

- Use **DINE (DeployIfNotExists)** policies to auto-remediate missing diagnostic settings
- Built-in policies exist for most resource types
- Custom policies for resources without built-in support
- Assign at management group level for organization-wide coverage
- Remediation tasks can backfill existing resources

## Best Practices

1. **Deploy as part of resource deployment** — include diagnostic settings in your Bicep/Terraform templates, not as an afterthought
2. **Use categoryGroup 'allLogs'** over individual categories — simpler, captures new categories automatically as they're added
3. **Archive to storage for compliance** — when retention beyond 2 years is required
4. **Event Hub for SIEM integration** — stream to Sentinel, Splunk, or other security tools
5. **Treat environments differently** — production gets full diagnostics, dev/test can use reduced categories
6. **Use Data Collection Transformations** to filter or transform data before ingestion (reduce cost)

## Common Gotchas

- Data may take up to 90 minutes to flow after creation
- Log Analytics tables are auto-created on first log receipt
- App Insights diagnostic setting destination cannot be the same workspace the App Insights resource uses
- Some resource types have resource-specific tables (preferred) vs AzureDiagnostics (legacy)
- Storage account diagnostics cannot be sent to the same storage account
