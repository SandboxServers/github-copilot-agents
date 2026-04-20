# Showback and Chargeback Patterns

Cost visibility (showback) and cost accountability (chargeback) are foundational FinOps capabilities.
Showback informs; chargeback bills. Implement showback first, then mature into chargeback.

## Showback vs. Chargeback

| Aspect | Showback | Chargeback |
|---|---|---|
| **Purpose** | Visibility — teams see what they spend | Accountability — teams pay for what they spend |
| **Financial impact** | Informational only, no budget transfer | Actual internal billing, hits department budgets |
| **Complexity** | Low — tag-based reporting is sufficient | High — requires finance integration, dispute resolution |
| **Maturity stage** | Crawl/Walk | Walk/Run |
| **Prerequisite** | Cost allocation (tagging, hierarchy) | Showback + agreed cost allocation rules |

## Cost Allocation Foundation

### Tagging Strategy for Allocation
- **Required tags:** `cost-center`, `team`, `application`, `environment`
- **Optional tags:** `project`, `business-unit`, `budget-code`
- Enforce via Azure Policy — `deny` effect for resources missing mandatory tags
- Enable tag inheritance in Cost Management to fill gaps without modifying resource tags
- Use cost allocation rules in Cost Management to redistribute shared costs (networking, security, monitoring)

### Hierarchy-Based Allocation
- Structure management groups → subscriptions → resource groups to mirror organizational cost boundaries
- Dedicate subscriptions per team or workload for natural cost isolation
- Use resource groups for project-level cost grouping within shared subscriptions

## Showback Implementation

### Getting Started
1. Enable Cost Management exports to a shared Storage Account
2. Build Power BI dashboards using the Cost Management connector
3. Publish weekly/monthly cost reports per team, application, and environment
4. Include actual vs. budget variance and trend lines
5. Highlight top-5 cost drivers and Advisor recommendations per team

### Showback Report Contents
- Amortized costs (spreads reservation/savings plan purchases across usage period)
- Cost by tag dimension: team, application, environment
- Month-over-month and quarter-over-quarter trends
- Untagged/unallocated cost highlighted as governance debt
- Advisor savings opportunities per team

## Chargeback Implementation

### Prerequisites
- Agreed cost allocation rules with all stakeholders (IT, Finance, business units)
- Defined approach for shared costs: proportional split, fixed allocation, or direct assignment
- Documented dispute resolution process
- Finance system integration path (ERP, SAP, internal billing)

### Shared Cost Strategies
| Strategy | When to Use | Example |
|---|---|---|
| **Proportional** | Usage-based services | Shared Log Analytics — split by ingestion volume per team |
| **Fixed split** | Platform services | Hub networking — 25% each to 4 business units |
| **Direct assignment** | Dedicated resources | Team-owned VMs billed directly |
| **Weighted allocation** | Complex shared services | Kubernetes cluster — split by namespace CPU/memory requests |

### Commitment Discount Allocation
- Amortize reservation and savings plan costs across benefiting resources
- Use Cost Management cost allocation rules to redistribute unused commitment costs
- Report commitment utilization per team — teams that drive underutilization should adjust

## FinOps Maturity Progression

| Stage | Showback | Chargeback |
|---|---|---|
| **Crawl** | Basic cost reports by subscription | No chargeback — awareness only |
| **Walk** | Tag-based showback per team/app | Informal chargeback, manual reconciliation |
| **Run** | Automated dashboards, anomaly alerts | Automated chargeback integrated with finance tools |

## Sources

- [Invoicing and chargeback — FinOps Framework](https://learn.microsoft.com/cloud-computing/finops/framework/manage/invoicing-chargeback)
- [Cost allocation introduction](https://learn.microsoft.com/azure/cost-management-billing/costs/cost-allocation-introduction)
- [Data analysis and showback](https://learn.microsoft.com/cloud-computing/finops/framework/understand/reporting)
- [Cost allocation capability](https://learn.microsoft.com/cloud-computing/finops/framework/understand/allocation)
- [Collecting and reviewing cost data](https://learn.microsoft.com/azure/well-architected/cost-optimization/collect-review-cost-data)
