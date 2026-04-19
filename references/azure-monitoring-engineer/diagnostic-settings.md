# Diagnostic Settings

## What They Do
- Route platform logs and metrics from Azure resources to destinations:
  - Log Analytics workspace (for querying and alerting)
  - Storage account (for archival and compliance)
  - Event Hub (for streaming to external systems)
  - Partner solutions
- Must be configured per resource — they are NOT automatic
- Data flows within 90 minutes of creation; tables auto-created on first log receipt

## Design Principles
- **Configure as part of deployment** — not as an afterthought. Include in Terraform/Bicep/ARM templates.
- **Collect only what you need** — each category costs money. Don't enable all categories "just in case"
- **Use transformations** to filter within categories if needed (Data Collection Transformations)
- **Treat non-production differently** — dev/test environments need less diagnostic data
- **App Insights gotcha**: diagnostic setting destination can't be the same workspace the App Insights resource is based on

## Common Categories to Consider
- **AllMetrics**: send platform metrics to Log Analytics for complex analysis (already available in Metrics Explorer for free)
- **Audit logs**: control plane operations — who did what when
- **Resource logs**: data plane operations — varies by service (e.g., SQL queries, storage access, key vault operations)
