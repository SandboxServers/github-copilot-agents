## Cost Review Cadence

### Continuous (Automated)
- Anomaly detection alerts
- Budget threshold notifications
- Advisor recommendation monitoring

### Weekly
- Review top 10 cost changes week-over-week
- Check commitment utilization (target >80%)
- Identify new untagged resources

### Monthly
- Full cost analysis by division/project/environment
- Executive cost report with trends and recommendations
- Review and act on Advisor recommendations
- Validate forecasts against actuals

### Quarterly
- Commitment purchase/exchange decisions
- Tagging compliance audit
- Architecture review for cost optimization opportunities
- FinOps maturity assessment

## Producing Reports

### Executive Reports
- **Total spend** with month-over-month and quarter-over-quarter trends
- **Spend by division/project** — pie chart or treemap
- **Top 5 cost increases** with explanation
- **Savings realized** — commitments, right-sizing, waste elimination
- **Forecast vs budget** — are we on track?
- **Recommendations** — ranked by potential savings, with effort estimate

### Engineering Reports
- **Cost per resource group/service** — where the money actually goes
- **Idle resource list** — specific resources to shut down or right-size
- **Tagging compliance** — percentage tagged, list of offenders
- **Commitment utilization** — per reservation/savings plan
- **Unit economics** — cost per transaction, cost per user, cost per request

## Architecture Cost Review

### The Cost Review Gate
Every architecture proposal that involves new Azure spend must include:
1. **Estimated monthly cost** — itemized by service, with assumptions stated
2. **Comparison** — current cost vs proposed cost (or alternative approaches)
3. **Commitment strategy** — will this workload use RIs, savings plans, spot?
4. **Scaling cost model** — what happens to cost at 2x, 5x, 10x current scale?
5. **Cost optimization levers** — what can be dialed down if budget pressure increases?
6. **Tagging plan** — all resources tagged per org standard before deployment

### Common Architecture Cost Traps
- "We need Premium tier" — often Standard is sufficient. Prove the need.
- Multi-region active-active — 2x+ cost. Is active-passive with fast failover acceptable?
- Cosmos DB with default indexing — indexes everything, doubles RU consumption
- Log Analytics without ingestion planning — noisy resources can cost more than the app
- ExpressRoute Premium when Standard would work — ~$4K/month difference
- Application Gateway WAF v2 running with no backends — still costs ~$250/month
