# Commitment Discounts

Commitment-based discounts are the primary lever for reducing Azure compute costs on stable workloads.
Always right-size and eliminate waste before purchasing commitments — discounts reduce rates, not waste.

## Discount Comparison

| Discount Type | Savings | Commitment | Flexibility | Cancellation |
|---|---|---|---|---|
| **Reservations** | 30–72% | 1 or 3 year | Instance size flexibility within family | Partial refund (12-month limit) |
| **Savings Plans** | 15–65% | 1 or 3 year | Any size/region/family (compute) | No cancellation, exchange to reservation |
| **Spot VMs** | Up to 90% | None | Eviction with 30s notice | N/A |
| **Dev/Test pricing** | ~55% off Windows/SQL | EA/MCA subscription | Dev/test workloads only | N/A |

## Reserved Instances

Best for stable, predictable workloads where SKU family and region are known.

### Scope Options
- **Shared** — applies across all subscriptions in billing account
- **Single subscription** — limited to one subscription
- **Resource group** — limited to one resource group
- **Management group** — applies across subscriptions in a management group

### Key Behaviors
- Instance size flexibility — reservation for D4s applies proportionally to D2s, D8s within same family
- Auto-renew option — enable for commitments that remain valid
- Exchange allowed — swap for different SKU/region (12-month value limit)
- Partial refund available — subject to 12-month cancellation limit per billing account
- Payment options — all upfront (most savings), monthly, or no upfront

### When to Buy
- Workload has run stable for 30+ days at consistent utilization
- No planned migration or decommission within commitment term
- Azure Advisor shows reservation recommendation with high confidence

## Savings Plans

Best when workload size, region, or service may change over the commitment term.

### Coverage
- **Compute Savings Plan** — broadest: VMs, App Service, Functions Premium, Container Instances, Dedicated Hosts
- **VM-specific Savings Plan** — narrower but deeper discount on VMs only

### Key Behaviors
- Spend-based commitment — commit to $/hour, Azure auto-applies to best discount
- No cancellation — but can exchange remaining value to a reservation
- Applies across regions and SKU families automatically
- Stacks with Azure Hybrid Benefit for additional savings

### When to Buy
- Workload sizes or regions may shift during commitment term
- Multi-service compute portfolio (VMs + App Service + Functions)
- Want set-and-forget without managing individual reservations

## Spot VMs

Best for interruptible, fault-tolerant workloads.

### Use Cases
- Batch processing and data pipelines
- CI/CD build agents
- Dev/test environments that tolerate interruption
- HPC and rendering workloads
- AKS spot node pools for non-critical workloads

### Key Behaviors
- Eviction with 30-second notice — application must handle graceful shutdown
- Pricing varies by region, SKU, and demand — set max price or use default (-1)
- No SLA — not suitable for production workloads requiring availability
- Savings up to 90% vs pay-as-you-go pricing

## Dev/Test Pricing

Available on EA and MCA subscriptions with Visual Studio subscribers.

### What's Discounted
- No Windows OS charges on VMs (~55% savings on Windows VMs)
- No SQL Server license charges on SQL VMs
- Reduced rates on select PaaS services (App Service, Logic Apps, etc.)
- Not available for production workloads — compliance requirement

## Decision Tree

1. **Is the workload interruptible?** → Spot VMs
2. **Is it a dev/test workload?** → Dev/Test pricing subscription
3. **Is workload stable, same SKU/region for 1–3 years?** → Reservation
4. **Is workload stable but size/region may change?** → Savings Plan
5. **Is workload short-lived or unpredictable?** → Pay-as-you-go

## Stacking Strategy

Reservations and Savings Plans can stack together for maximum coverage:

1. **Buy reservations first** for the stable base — highest discount rate
2. **Buy savings plans** for the variable portion — flexibility to cover shifting workloads
3. Reservations apply first, then savings plans cover remaining eligible compute
4. Azure Hybrid Benefit applies on top of both for Windows/SQL workloads
5. Monitor combined utilization — target 80%+ across all commitments

### Utilization Monitoring
- Set commitment utilization alerts — notify if utilization drops below 80%
- Review Azure Advisor recommendations weekly for purchase/exchange suggestions
- Check utilization reports in Cost Management → Reservations and Savings Plans
- Exchange or refund underperforming reservations before value decays
