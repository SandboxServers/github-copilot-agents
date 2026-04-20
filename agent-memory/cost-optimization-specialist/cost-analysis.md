# Cost Analysis and Reporting

Cost analysis transforms raw billing data into actionable insights. Regular analysis identifies
trends, anomalies, and optimization opportunities before they become budget problems.

## Cost Management Views

### Built-in Capabilities
- Filter and group by resource group, tag, service name, location, meter category
- Daily or monthly granularity — daily for troubleshooting, monthly for trends
- Accumulated or actual cost views — accumulated shows running total toward budget
- Save custom views for repeated analysis — pin to Azure dashboard for quick access
- Share views with stakeholders via direct link or dashboard

### Recommended Custom Views
- **By environment** — group by Environment tag to compare prod vs dev vs staging spend
- **By team** — group by CostCenter or Owner tag for showback/chargeback
- **By service** — identify which Azure services drive the most cost
- **By location** — spot unexpected cross-region resource placement
- **Daily trend** — identify cost spikes and correlate with deployments or incidents

## Common Analyses

### Month-Over-Month Trend
- Compare current month spend to previous month at same point in time
- Identify growth rate — is spend increasing faster than business metrics?
- Flag any service category growing > 20% month-over-month for investigation

### Cost by Environment
- Production should be the majority of spend (typically 60–80%)
- Dev/test exceeding production is a red flag — investigate immediately
- Staging should be minimal — spin up for testing, tear down after

### Top Expensive Resources
- Identify the top 10 most expensive individual resources monthly
- Validate each is correctly sized and actively used
- Challenge any single resource consuming > 15% of subscription budget

### Unused Resources
- Zero-traffic App Service plans — no requests in 30 days
- Unattached managed disks — not associated with any VM
- Idle public IP addresses — not assigned to any resource
- Stopped VMs with Premium disks — still paying for disk even when VM is deallocated
- Unused Application Gateways — running with no backend pools configured

## Anomaly Detection

### Azure Cost Management Anomaly Alerts
- ML-based detection identifies unexpected cost spikes or drops automatically
- Configurable scope — subscription, resource group, or service level
- Daily evaluation with email notification when anomaly is detected
- Includes root cause analysis — which resources caused the anomaly

### Custom Threshold Alerts
- Set budget alerts at specific dollar amounts for predictable thresholds
- Use forecast alerts to catch trends before they hit the budget
- Combine anomaly detection (unexpected) with budget alerts (expected limits)

## Exports and External Reporting

### Scheduled Exports
- Configure daily, weekly, or monthly export to Azure Storage account
- CSV format compatible with Power BI, Excel, and third-party tools
- FOCUS format available for FinOps Foundation-compliant tools
- Use Azure Data Factory or Synapse for advanced transformation and analysis

### Power BI Integration
- Cost Management connector provides pre-built reports
- Customize for executive dashboards — spend by business unit, trend, forecast
- Combine with operational data for unit economics (cost per transaction, per user)

## Cost Allocation

### Shared Resource Challenges
- Hub VNet, Azure Firewall, and Log Analytics workspace serve multiple workloads
- ExpressRoute circuit shared across subscriptions
- Shared App Service plans hosting multiple applications

### Allocation Methods
- **Tag-based** — allocate by tag proportions (e.g., 40% Team A, 60% Team B)
- **Usage-based** — split by actual consumption metrics (data transfer, log volume)
- **Fixed percentage** — agreed-upon split regardless of actual usage
- **Cost allocation rules** — native Cost Management feature for redistributing shared costs
- Document allocation methodology — stakeholders must agree on the approach

## Executive Reporting

### Monthly Cost Report Template
1. **Total spend** — current month, previous month, year-over-year comparison
2. **Trend** — 6-month spend trend with projected next month
3. **Top changes** — 5 largest cost increases/decreases with root cause explanation
4. **Savings achieved** — commitment discounts, right-sizing, waste elimination savings this month
5. **Recommendations** — top 5 optimization opportunities ranked by potential savings
6. **Budget status** — actual vs budget for each business unit/subscription
7. **Commitment utilization** — reservation and savings plan utilization rates

### Delivery Cadence
- Monthly executive summary to leadership
- Weekly top-changes summary to engineering leads
- Real-time dashboards available to all team members
- Quarterly deep-dive with architecture review and commitment planning
