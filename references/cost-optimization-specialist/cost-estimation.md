# Cost Estimation

Accurate cost estimation prevents budget surprises and builds trust with stakeholders.
Every architecture proposal should include a cost estimate before deployment approval.

## Azure Pricing Calculator

The primary tool for pre-deployment cost estimation.

### Usage
1. Navigate to azure.microsoft.com/pricing/calculator
2. Add each Azure service needed for the workload
3. Configure SKU, region, quantity, and expected usage volume for each service
4. Review monthly and annual cost estimate
5. Export estimate as Excel — share with stakeholders and attach to architecture proposals

### Best Practices
- Use the same region for all services to minimize data transfer estimates
- Include all supporting services (Key Vault, DNS, monitoring, backup)
- Model both low and high usage scenarios for variable workloads
- Save estimates for comparison after deployment

## TCO Calculator

For comparing on-premises costs to Azure — used in migration business cases.

### What to Include
- **Compute** — server hardware, virtualization, depreciation
- **Storage** — SAN, NAS, backup infrastructure
- **Networking** — switches, routers, bandwidth, WAN links
- **Licensing** — OS, database, middleware licenses and support
- **Facilities** — power, cooling, rack space, physical security
- **Labor** — system administration, hardware maintenance, patching

### Output
- 3-year and 5-year TCO comparison
- Cost breakdown by category
- Use as justification for cloud migration business cases

## Estimation Process

### Step-by-Step
1. **List all Azure services** needed for the architecture
2. **Estimate SKU and quantity** — use minimum viable size, not maximum
3. **Estimate data transfer** — often the most underestimated cost component
4. **Estimate storage growth** — project data volume at 6 months and 12 months
5. **Add 20% buffer** — for unknowns, unexpected usage, and services missed in estimation
6. **Apply commitment discounts** — reduce estimate for workloads eligible for RIs or Savings Plans
7. **Document assumptions** — every estimate should list what was assumed about usage patterns

## Commonly Underestimated Costs

These cost components are most frequently missed or underestimated in proposals:

| Component | Why It's Missed | Impact |
|---|---|---|
| **Data transfer (egress)** | Developers don't think about network costs | Can be 10–20% of total bill |
| **Log Analytics ingestion** | Logging volume is hard to predict | $2.76/GB ingested, verbose apps generate GBs/day |
| **Private Link data processing** | Per-GB charge on private endpoint traffic | Significant for high-throughput services |
| **Support plan** | Not included in resource pricing | $100–$1,000+/month depending on plan |
| **Backup storage** | Default retention generates growing costs | Geo-redundant backup storage especially |
| **DNS queries** | High-traffic apps generate millions of queries | $0.40 per million queries for Azure DNS |
| **Diagnostic settings** | Enabled broadly, ingestion cost not considered | Can double monitoring costs |

## Cost Gates

### Architecture Review Cost Gate
Every architecture proposal involving new Azure spend must include:

1. **Itemized monthly cost estimate** — by service, with SKU and quantity assumptions
2. **Comparison** — current cost vs proposed cost, or cost of alternatives considered
3. **Commitment strategy** — will this workload use RIs, Savings Plans, or Spot?
4. **Scaling cost model** — what happens to cost at 2x, 5x, 10x current scale?
5. **Cost optimization levers** — what can be scaled down or switched off under budget pressure?
6. **Tagging plan** — all resources tagged per organizational standard before deployment

### Post-Deployment Validation
- Compare actual spend to estimate after 30 days of production usage
- Investigate any variance > 20% — update estimation models with learnings
- Adjust forecasts and budgets based on actual observed costs
- Feed learnings back into estimation process for future proposals
