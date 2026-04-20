# Cost Governance

Cost governance establishes the organizational guardrails that prevent cloud cost overruns.
Without governance, optimization is reactive — governance makes cost control systematic and preventive.

## Budget Hierarchy

Budgets should mirror the organizational and technical structure:

| Level | Scope | Owner | Purpose |
|---|---|---|---|
| **Management group** | Business unit / division | Finance + VP Engineering | Top-level spend envelope |
| **Subscription** | Workload or environment | Service owner | Workload-level spend control |
| **Resource group** | Application component | Team lead | Granular component tracking |

### Alert Thresholds
Configure alerts at progressive thresholds for early warning:

- **50%** — informational, mid-period checkpoint
- **75%** — warning, review spending trajectory
- **90%** — critical, investigate and take action
- **100%** — budget exceeded, escalate to management
- **110%** — over budget, trigger automated remediation via action groups

### Forecast Alerts
- Set forecast alerts at 110% — Azure projects end-of-period spend based on current trend
- Forecast alerts are proactive — trigger before actual spend exceeds budget
- More valuable than actual-spend alerts for preventing overruns

### Action Groups
- Email and SMS to finance leads and engineering owners
- Microsoft Teams / Slack webhook for real-time visibility
- Azure Function or Logic App for automated remediation (scale down, stop VMs)
- Link directly to Cost Analysis filtered to the affected scope

## Tagging Strategy

Tags are the foundation of cost allocation, showback, and chargeback.

### Mandatory Tags

| Tag | Purpose | Example | Enforcement |
|---|---|---|---|
| `CostCenter` | Finance allocation | `CC-12345` | Deny if missing |
| `Owner` | Accountability | `team-platform@contoso.com` | Deny if missing |
| `Environment` | Dev/test/staging/prod | `Production` | Deny if missing |
| `Application` | Application grouping | `MeridianConsole` | Deny if missing |

### Recommended Tags

| Tag | Purpose | Example |
|---|---|---|
| `Project` | Project-level tracking | `ProjectAlpha` |
| `Department` | Org-level rollup | `Engineering` |
| `ExpirationDate` | Temporary resource cleanup | `2026-06-30` |
| `CreatedBy` | Audit trail | `terraform` |

### Enforcement Approach
1. **Azure Policy deny** — block deployments missing mandatory tags on resource groups and subscriptions
2. **Tag inheritance** — enable in Cost Management to propagate RG/subscription tags to child resources
3. **Azure Policy inherit** — copy tags from resource group to resources at deploy time
4. **Compliance KPI** — track tagging compliance percentage, target 95%+
5. **Regular audits** — weekly report of untagged resources to resource owners

## Azure Policy for Cost Control

Policy is the enforcement mechanism for cost governance decisions.

### Common Cost Policies
- **Deny expensive SKUs** — restrict allowed VM sizes to approved list per environment
- **Deny unapproved regions** — limit deployments to approved regions (reduces data transfer costs)
- **Audit untagged resources** — flag resources missing mandatory cost tags
- **Deny public IPs** — reduce unnecessary networking costs and attack surface
- **Enforce auto-shutdown** — require auto-shutdown tags on non-production VMs
- **Restrict Premium storage** — audit or deny Premium SSD in dev/test environments

### Policy Assignment Strategy
- Assign deny policies at management group level for organization-wide enforcement
- Use audit mode first — review compliance before switching to deny
- Exempt production subscriptions from restrictive SKU policies when justified
- Review policy compliance weekly as part of cost governance cadence

## Cost Management + Billing

The primary tool for ongoing cost visibility and analysis.

### Key Features
- **Cost analysis views** — filter by resource group, tag, service, location, meter
- **Exports** — scheduled export to storage account (daily/weekly/monthly CSV)
- **Power BI integration** — Cost Management connector for executive dashboards
- **Anomaly detection** — ML-based detection of unexpected cost spikes or drops
- **Cost allocation rules** — redistribute shared costs across teams/subscriptions

## FinOps Culture

Governance works only when backed by organizational culture.

### Principles
- **Teams own their cloud spend** — engineering teams are accountable for their resource costs
- **Showback first** — provide visibility into costs before implementing chargeback
- **Then chargeback** — allocate actual costs to business units once tagging and processes mature
- **Monthly cost review cadence** — dedicated meeting to review spend, trends, and actions
- **Engineer awareness** — every engineer understands the cost impact of their architecture decisions
- **Celebrate savings** — recognize teams that optimize effectively, not just those that spend less
