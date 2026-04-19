## Landing Zone Integration

### Platform Landing Zone
The backbone of the Azure environment:
- Management group hierarchy with Azure Policy enforcement
- Dedicated subscriptions for connectivity, identity, management, security
- Shared services that all application landing zones consume

### Application Landing Zones
Where workloads live:
- Pre-provisioned through subscription vending automation
- Nested under management groups (`Online`, `Corp`) for policy inheritance
- One or more subscriptions per workload per environment

### Three Management Approaches
| Approach | Who Operates | When to Use |
|---|---|---|
| **Central team** | Central IT fully operates landing zone | Small org, early maturity |
| **Application team** | Platform team delegates to workload team | Mature teams, high autonomy |
| **Shared** | Central IT manages underlying service, app teams manage apps | AKS, AVS, shared platforms |
