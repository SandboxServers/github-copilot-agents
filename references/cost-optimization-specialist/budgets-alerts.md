## Budgets and Alerts

### Budget Configuration
- Set budgets at **subscription** and **resource group** level
- Create per-service budgets for high-cost services (Cosmos DB, Log Analytics, AKS)
- Minimum alert thresholds:
  - **90%** (ideal spend) — informational
  - **100%** (target budget) — warning
  - **110%** (over budget) — critical, trigger action group

### Alert Types
1. **Budget alerts** — actual spend exceeds threshold (reactive)
2. **Forecast alerts** — projected spend will exceed threshold (proactive) — set at 110%
3. **Anomaly alerts** — unexpected cost spikes detected automatically
4. **Commitment utilization alerts** — RI/savings plan utilization drops below threshold

### Action Groups
- Email/SMS to finance and engineering leads
- Logic App / Azure Function for automated remediation (scale down, shut down)
- Teams/Slack notification for immediate visibility
- Link to Cost Analysis filtered to the anomalous scope
