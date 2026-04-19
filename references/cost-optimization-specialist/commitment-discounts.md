## Commitment-Based Discounts

### The Right Sequence
1. **Right-size first** — remove unused/oversized resources before buying discounts (discounts reduce rates, not waste)
2. **Exchange underutilized reservations** — reassign to better-fit configurations
3. **Trade-in underutilized reservations for savings plans** — when usage is variable
4. **Purchase new reservations** — only for stable, well-understood workloads
5. **Purchase new savings plans** — flexible, spend-based commitments sized to clean baseline

### Reserved Instances vs Savings Plans vs Spot vs PayGo

| Aspect | Reserved Instances | Savings Plans | Spot VMs | Pay-As-You-Go |
|---|---|---|---|---|
| **Commitment** | Specific SKU + region, 1 or 3 years | $/hour on compute, 1 or 3 years | None | None |
| **Savings** | Up to 72% | Up to 65% | Up to 90% | Baseline (0%) |
| **Flexibility** | Low — locked to SKU family + region | High — applies across SKUs, regions, services | None — can be evicted with 30s notice | Full |
| **Best for** | Stable, predictable workloads | Dynamic workloads, multi-region, multi-SKU | Batch, dev/test, interruptible workloads | Unpredictable, short-term, experimenting |
| **Risk** | Waste if workload changes | Lower — auto-applies to best match | Eviction at any time | None |
| **Payment** | Upfront, monthly, or no upfront | Monthly (no additional cost) | Variable spot pricing | Per-hour/per-second |

### Modeling Commitments
- View **Advisor recommendations** for RI and savings plan purchase suggestions based on 7-30 day usage
- Use the **reservation purchase experience** in portal — filter and compare recommendations
- Check **utilization** after purchase — set alerts if utilization drops below 80%
- Enable **auto-renewal** for commitments that remain valid
- **Scope** commitments appropriately: single subscription vs shared vs management group
- Calculate actual savings using Benefit Recommendations API and Power BI reports

### Azure Hybrid Benefit
- Use existing Windows Server or SQL Server licenses on Azure VMs
- Significant savings on top of RI/savings plan discounts
- Requires Software Assurance or qualifying subscription licenses
- Apply at VM level or centrally manage at subscription level
