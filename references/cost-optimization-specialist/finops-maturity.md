# FinOps Maturity Model

FinOps maturity represents the progression from basic cost visibility to fully operationalized
cloud financial management. Organizations advance through three stages as practices mature.

## Crawl — Visibility

The foundation: understand where money is going and establish baseline practices.

### Key Activities
- Enable Azure Cost Management across all subscriptions
- Implement mandatory tagging strategy (CostCenter, Owner, Environment, Application)
- Set budgets at subscription and resource group level with alert thresholds
- Create basic cost reports — total spend, spend by subscription, spend by service
- Identify the top 10 most expensive resources and validate they're correctly sized
- Assign cost owners for each subscription or resource group

### Success Criteria
- 90%+ tagging compliance on mandatory tags
- Budgets configured for all production subscriptions
- Monthly cost report generated and reviewed by leadership
- Top 10 cost items validated and documented

## Walk — Optimization

Active optimization: reduce waste, purchase commitments, automate savings.

### Key Activities
- **Right-sizing automation** — act on Azure Advisor recommendations weekly
- **Commitment discounts** — purchase reservations and savings plans for stable workloads
- **Auto-shutdown** — implement start/stop schedules for all non-production compute
- **Waste elimination** — monthly sweep for orphaned resources, unattached disks, idle IPs
- **Showback to teams** — publish cost reports per team, making spend visible to engineering
- **Log Analytics optimization** — reduce over-ingestion, implement Basic Logs for noisy tables
- **Cost-aware architecture reviews** — include cost estimates in every design proposal

### Success Criteria
- 80%+ commitment utilization across reservations and savings plans
- Non-production compute running on schedule (65%+ savings vs 24/7)
- Monthly waste elimination saving measurable dollars
- Teams receive and review their own cost reports

## Run — Operations

Fully operationalized FinOps: cost is a first-class engineering concern.

### Key Activities
- **Chargeback** — actual costs allocated to business units based on usage
- **Real-time anomaly detection** — immediate alerts on unexpected cost changes
- **Cost-aware architecture decisions** — cost is a design criterion alongside performance and reliability
- **Dedicated FinOps team** — central team coordinating cloud financial management
- **Unit economics** — track cost per transaction, per user, per API call as business metrics
- **Automated remediation** — action groups scale down or shut off resources when thresholds breach
- **Continuous optimization** — reservations reviewed monthly, right-sizing automated, waste near zero

### Success Criteria
- Cost per unit metric tracked and trending down over time
- Waste percentage < 5% of total spend
- Chargeback operational with business unit acceptance
- FinOps cadence embedded in engineering culture

## Key Metrics

Track these metrics to measure FinOps maturity and progress.

| Metric | Description | Target |
|---|---|---|
| **Cost per environment** | Spend breakdown by prod/dev/staging | Dev < 30% of prod |
| **Cost per team/service** | Tag-based allocation per team | Trending with business growth |
| **Unit cost** | Cost per API call, per user, per transaction | Declining over time |
| **Waste percentage** | Idle/orphaned resource cost as % of total | < 5% |
| **Commitment coverage** | % of eligible compute covered by RI/SP | > 80% |
| **Commitment utilization** | % of purchased commitments actively used | > 80% |
| **Savings achieved** | Monthly $ saved from optimization actions | Growing quarter-over-quarter |
| **Tagging compliance** | % of resources with mandatory tags | > 95% |

## Organizational Structure

### FinOps Team (Central)
- 1–3 dedicated FinOps practitioners depending on cloud spend scale
- Reports to finance or engineering leadership (or shared)
- Owns tooling, reporting, governance policies, and commitment purchases
- Facilitates monthly cost reviews and quarterly planning

### Cloud Cost Champions
- One engineer per team designated as cost champion
- Attends monthly FinOps review, brings back action items to their team
- First point of contact for cost questions within their domain
- Responsible for acting on Advisor recommendations for their resources

### Executive Sponsor
- VP or Director level who champions FinOps investment and culture
- Reviews monthly executive cost report
- Approves commitment purchases above threshold
- Escalation point for cross-team cost disputes

## Tools

| Tool | Scope | Best For |
|---|---|---|
| **Azure Cost Management** | Native Azure | Primary tool for all Azure cost analysis and governance |
| **FinOps Foundation FOCUS** | Multi-cloud | Standardized cost data format for cross-cloud analysis |
| **Power BI** | Reporting | Custom executive dashboards and trend analysis |
| **Kubecost** | Kubernetes | AKS namespace and workload cost allocation |
| **CloudHealth (VMware)** | Multi-cloud | Enterprise multi-cloud cost management |
| **Spot.io (NetApp)** | Compute optimization | Automated spot instance management and optimization |
| **Azure FinOps Hubs** | Advanced analytics | Scalable cost analytics via Fabric/Data Explorer |
