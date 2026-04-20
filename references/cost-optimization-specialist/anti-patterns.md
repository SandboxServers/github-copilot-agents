# Cost Optimization Anti-Patterns

Common mistakes that undermine Azure cost management efforts.
Recognizing these patterns is the first step — each has a concrete remedy.

## Governance Anti-Patterns

### No Tagging Strategy
- Cannot attribute costs to teams, projects, or environments
- Shared costs become invisible and unaccountable
- **Fix:** Enforce mandatory tags (`environment`, `cost-center`, `team`, `application`) via Azure Policy with `deny` effect
- Use tag inheritance in Cost Management to fill gaps in cost data without modifying resources

### No Budgets or Alerts
- Surprise bills at end of month with no early warning
- No one accountable until finance raises the alarm
- **Fix:** Create budgets at subscription and resource group level with alerts at 50%, 75%, 90%, 100% thresholds
- Enable forecast alerts at 110% to catch trending overruns before they happen
- Set up anomaly detection for unexpected cost spikes

### No Regular Cost Review Cadence
- Costs drift unchecked between quarterly reviews
- Optimization opportunities expire (Advisor recommendations have a shelf life)
- **Fix:** Weekly team-level reviews of Advisor recommendations, monthly leadership reviews of cost trends
- Use Cost Management exports and Power BI for trend analysis

## Commitment Discount Anti-Patterns

### Buying Reservations Without Analyzing Usage
- Purchased reservations go underutilized — use-it-or-lose-it model
- Wrong VM family or region locks in waste for 1–3 years
- **Fix:** Analyze 30+ days of usage in Advisor or Cost Management before purchasing
- Start with 1-year terms to limit risk, verify utilization before extending to 3-year

### Single Reservation Strategy
- All-in on reservations ignores workload flexibility needs
- Changing VM families or regions strands reservation investment
- **Fix:** Mix reservations (for stable, predictable workloads) with savings plans (for flexible compute)
- Savings plans cover broader compute services across regions and VM families
- Use reservations for databases and storage where commitment is clear

## Resource Sizing Anti-Patterns

### Over-Provisioned Resources "Just in Case"
- Premium SKUs and oversized VMs deployed for workloads that never scale
- 8-core VMs running at 5% CPU utilization
- **Fix:** Monitor for 2+ weeks, then right-size using Advisor recommendations
- Enable autoscale where supported — scale to zero for non-production

### Dev/Test Environments on Production SKUs
- Standard_D16s_v5 VMs running unit tests
- Premium SSD and geo-redundant storage for throwaway data
- **Fix:** Use B-series burstable VMs for dev/test workloads
- Use Standard SSD or Standard HDD for non-production storage
- Apply Dev/Test subscription pricing (up to 55% discount on Windows VMs)

### Not Using Auto-Shutdown for Dev/Test VMs
- Development VMs running 24/7 when used 8–10 hours/day
- ~65% of compute cost is pure waste on nights and weekends
- **Fix:** Enable auto-shutdown schedules on all non-production VMs
- Use Azure DevTest Labs or start/stop automation for predictable schedules

## Advisory Anti-Patterns

### Ignoring Advisor Cost Recommendations
- Free, actionable recommendations sit unreviewed for months
- Advisor estimates monthly savings per recommendation — money left on the table
- **Fix:** Review Advisor weekly, assign owners to recommendations, track implementation
- Use the Cost Optimization Workbook for cross-subscription visibility

### Licensing Waste
- Unused or underused Microsoft 365, SQL Server, or Windows Server licenses
- Not activating Azure Hybrid Benefit for eligible workloads
- **Fix:** Audit license assignments quarterly against actual usage
- Enable Azure Hybrid Benefit on VMs and SQL to save up to 40–55% on compute

### Not Using Dev/Test Subscription Pricing
- Running dev/test workloads on standard Pay-As-You-Go subscriptions
- Missing discounted rates on Windows VMs, SQL, App Service, and more
- **Fix:** Create Enterprise Dev/Test or Pay-As-You-Go Dev/Test subscriptions for non-production
- No SLA, but significant pricing discounts for eligible services

## Timing Anti-Patterns

### Optimizing Prematurely Before Architecture Is Stable
- Buying 3-year reservations during a proof-of-concept phase
- Right-sizing during active development when load patterns are unknown
- **Fix:** Wait until architecture stabilizes and usage patterns are clear (typically 2–4 weeks of steady-state)
- Focus early efforts on governance (tagging, budgets, alerts) — optimize compute and commitments later

## Sources

- [Cost Management best practices](https://learn.microsoft.com/azure/cost-management-billing/costs/cost-mgt-best-practices)
- [FinOps Framework principles](https://learn.microsoft.com/cloud-computing/finops/framework/finops-framework)
- [Well-Architected Cost Optimization principles](https://learn.microsoft.com/azure/well-architected/cost-optimization/principles)
- [Cost optimization spending guardrails](https://learn.microsoft.com/azure/well-architected/cost-optimization/set-spending-guardrails)
