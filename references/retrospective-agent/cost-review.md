## Cost Review

### Purpose
Reviewing actual costs against estimates identifies estimation gaps, uncovers optimization
opportunities, and builds institutional knowledge for more accurate future budgeting.
Cost surprises are a planning failure, not an execution failure.

### Actual vs Estimated Cost
Compare at multiple levels of granularity:
- Total engagement cost vs total budget
- Cost per workstream (infrastructure, application, security, testing)
- Cost per environment (development, staging, production, shared services)
- Cost per resource category (compute, storage, networking, security, monitoring)
- Monthly burn rate vs projected burn rate

### Cost Per Environment
| Environment | Estimated Monthly | Actual Monthly | Variance | Notes |
|---|---|---|---|---|
| Development | | | | Often over-provisioned |
| Staging | | | | Should mirror production scale-down |
| Production | | | | Baseline for capacity planning |
| Shared services | | | | Monitoring, key vault, DNS, etc. |
| Disaster recovery | | | | Often forgotten in estimates |

### Common Unexpected Costs
These categories frequently surprise teams that did not account for them:
- **Data transfer (egress)** — cross-region, cross-vnet, and internet egress charges
- **Premium features** — premium storage, premium tier services enabled by default
- **Over-provisioned resources** — VMs sized for peak but running at 10% utilization
- **Idle resources** — dev/test environments running 24/7 instead of scheduled shutdown
- **Transaction costs** — storage transactions, API calls, message queue operations
- **Logging and monitoring** — Log Analytics ingestion, Application Insights sampling
- **DNS and networking** — private DNS zones, NAT gateway, load balancer rules
- **License costs** — third-party tools, marketplace images, support plans

### Cost Optimization Opportunities
Document opportunities discovered during the engagement:
- Right-sizing recommendations based on actual utilization data
- Reserved instance or savings plan candidates (stable, predictable workloads)
- Auto-scaling configurations that could reduce off-peak costs
- Storage tier optimization (hot to cool to archive based on access patterns)
- Spot instance candidates for fault-tolerant batch workloads
- Consolidation opportunities (shared resources across environments)
- Scheduled start/stop for non-production environments
- Tag-based cost allocation gaps that prevent accurate chargeback

### Budget Adherence
- Was the engagement completed within the approved budget?
- If over budget: at what point was the overrun identified? Was it escalated?
- If under budget: was scope reduced, or were estimates conservative?
- Were there formal budget checkpoints during the engagement?
- Did cost forecasts update as scope changed?

### Recommendations for Future Engagements
Based on this cost review, document actionable recommendations:
- Adjust cost estimation templates with correction factors for commonly missed items
- Add specific line items for categories that were previously overlooked
- Establish cost alert thresholds at 50%, 75%, and 90% of budget
- Require cost review at mid-engagement, not just at the end
- Include egress modeling in architecture decisions from the start
- Define environment shutdown schedules as part of project setup
- Use Azure Cost Management budgets and anomaly detection from day one

### Cost Review Checklist
- [ ] Actual vs estimated comparison completed at workstream level
- [ ] Unexpected costs identified and categorized
- [ ] Per-environment cost breakdown documented
- [ ] Optimization opportunities listed with estimated savings
- [ ] Budget adherence summary prepared for stakeholders
- [ ] Recommendations captured for estimation template updates
- [ ] Cost data added to organizational knowledge base for benchmarking
