# Cost Optimization Best Practices

Proven patterns for managing Azure spend effectively.
Implement in order: governance first, then right-sizing, then commitment discounts.

## Tagging Strategy

- Apply mandatory tags on every resource: `environment`, `team`, `cost-center`, `application`
- Enforce via Azure Policy with `deny` effect — no tag, no deployment
- Enable tag inheritance in Cost Management to propagate subscription and resource group tags to cost data
- Audit tag compliance weekly — treat untagged resources as governance debt
- Use `project` and `workload` tags for granular cost allocation and showback/chargeback

## Right-Sizing

- Monitor resource utilization for 2+ weeks before making sizing decisions
- Use Azure Advisor cost recommendations — refreshed daily, includes estimated monthly savings
- Target VMs with < 15% average CPU as right-sizing candidates
- Resize in place when possible (same VM family) to avoid redeployment
- Review App Service plans, database tiers, and container allocations — not just VMs
- Re-evaluate after each major release or traffic pattern change

## Commitment Discounts

- **Reservations** for stable, predictable workloads (VMs, SQL, Cosmos DB, Storage)
  - Analyze 30+ days of usage before purchasing
  - Start with 1-year terms; extend to 3-year after proving utilization
  - Use shared scope to maximize flexibility across subscriptions
- **Savings plans** for flexible compute (varying VM families, regions, or services)
  - Covers VMs, App Service, Container Instances, and Azure Functions Premium
  - Lower discount than reservations but broader applicability
- **Mix both:** Reservations for databases and known compute, savings plans for variable workloads
- Monitor utilization weekly — unused commitments are wasted money

## Auto-Scale and Scale-to-Zero

- Enable autoscale on every service that supports it: VMSS, App Service, AKS, Container Apps
- Configure scale-to-zero for non-production AKS node pools and Container Apps
- Use scheduled scaling for predictable traffic patterns (business hours, batch windows)
- Set minimum instances to 0 where SLA permits — pay nothing during idle periods

## Dev/Test Optimization

- **Auto-shutdown:** Configure on all non-production VMs (save 65%+ on nights/weekends)
- **B-series VMs:** Burstable instances ideal for dev/test at 40–60% lower cost than D-series
- **Dev/Test subscription pricing:** Up to 55% discount on Windows VMs, reduced rates on PaaS services
- **Lower-tier storage:** Standard SSD or HDD for non-production — no need for Premium in dev
- **Reduce redundancy:** LRS instead of GRS/ZRS for ephemeral dev/test data

## Budget Alerts

- Set budget alerts at **50%, 75%, 90%, 100%** of monthly budget for progressive early warning
- Enable **forecast alerts at 110%** to catch trending overruns before they hit
- Configure **anomaly detection** for unexpected cost spikes from autogrow, scaling events, or misconfigurations
- Route alerts to both engineering teams (for action) and finance (for visibility)
- Create budgets at multiple scopes: subscription, resource group, and per-team where possible

## Regular Cost Reviews

- **Weekly:** Engineering team reviews Advisor recommendations, acts on quick wins
- **Monthly:** Leadership reviews cost trends, budget variance, and commitment utilization
- **Quarterly:** Full FinOps review — evaluate commitment strategy, renegotiate EA terms if applicable
- Track savings achieved vs. target — make cost optimization a measured KPI

## Cost Management Exports and Analysis

- Configure Cost Management exports to Storage Account for detailed offline analysis
- Use Power BI with the Cost Management connector for custom dashboards and trend reporting
- Export amortized costs to accurately reflect reservation and savings plan impact
- Join cost data with resource metadata for rich allocation and chargeback reporting

## Spot VMs for Batch Workloads

- Use Spot VMs for fault-tolerant, stateless batch processing (60–90% savings)
- Implement checkpoint/restart logic in batch jobs to handle 30-second eviction notice
- Combine with VMSS Spot priority for automatic capacity management
- Avoid Spot for latency-sensitive or stateful workloads — no SLA guarantees

## Storage Lifecycle Policies

- Configure lifecycle management rules for automatic tiering: Hot → Cool → Archive
- Move blobs to Cool tier after 30 days of no access, Archive after 90 days
- Set expiration rules for logs, diagnostics, and temporary data
- Use Blob inventory reports to identify access patterns before setting policies
- Enable last access time tracking to make data-driven tiering decisions

## Sources

- [Cost Management best practices](https://learn.microsoft.com/azure/cost-management-billing/costs/cost-mgt-best-practices)
- [Well-Architected Cost Optimization pillar](https://learn.microsoft.com/azure/well-architected/cost-optimization/principles)
- [FinOps Framework capabilities](https://learn.microsoft.com/cloud-computing/finops/framework/finops-framework)
- [Cost alerts and spending guardrails](https://learn.microsoft.com/azure/well-architected/cost-optimization/set-spending-guardrails)
- [Collecting and reviewing cost data](https://learn.microsoft.com/azure/well-architected/cost-optimization/collect-review-cost-data)
