## Engagement Sizing

### Sizing Tiers

| Tier | Specialists | Duration | Typical Scope |
|------|------------|----------|---------------|
| **Small** | 1-2 specialists | 1-2 weeks | Single-domain work — one service configuration, one pipeline, one policy set. Usually one division plus shared services. |
| **Medium** | 3-5 specialists | 2-4 weeks | Multi-domain work — crosses two or more divisions. Network + compute, identity + platform, or full DevOps pipeline buildout. |
| **Large** | 5+ specialists | 4+ weeks | Full engagement — all four divisions active, complex dependencies, multiple workstreams running in parallel. Greenfield deployments, major migrations, platform overhauls. |

### Sizing Factors

**Greenfield vs Brownfield**:
- Greenfield starts clean but requires building everything — landing zones, identity, networking, governance. More predictable but more work upfront.
- Brownfield works within existing constraints — existing VNets, existing RBAC, existing policies. Less total work but more discovery, more surprises, and more risk of breaking what already works.
- Brownfield typically adds 30-50% complexity over greenfield for the same scope because of discovery and constraint navigation.

**Compliance Requirements**:
- Each compliance framework (SOC2, HIPAA, PCI-DSS, FedRAMP, GDPR) adds scope to every phase.
- Compliance increases the security review depth, constrains architecture choices, requires additional documentation, and often mandates specific configurations.
- A single compliance framework adds roughly one size tier — a "small" engagement with HIPAA becomes a "medium."

**Multi-Region Deployment**:
- Multi-region doubles networking complexity — peering, DNS, traffic routing, failover.
- Multi-region adds data residency considerations, cross-region replication design, and disaster recovery testing.
- Budget typically increases 40-80% over single-region for the same workload.

**Budget Constraints**:
- Limited budgets shape scope — if the budget only supports a small engagement, scope accordingly rather than cutting corners on a medium engagement.
- The cost analyst participates in scoping to ensure the engagement size matches the budget reality.
- It is better to deliver a well-executed small engagement than a half-finished medium one.

### Sizing Anti-Patterns

**Under-sizing a complex engagement**: Trying to fit a large engagement into a medium timeline. Results in skipped gates, incomplete documentation, and technical debt. If the scope requires 5+ specialists and 4+ weeks, size it accordingly.

**Over-sizing a simple engagement**: Assigning 5 specialists to change a firewall rule. Not every request needs all four divisions. Match the team to the actual scope.

**Ignoring compliance multiplier**: Scoping a HIPAA-compliant deployment as if compliance adds no work. It does — to every phase, every gate, and every deliverable.

**Brownfield optimism**: Assuming existing infrastructure is well-documented and well-configured. Budget discovery time. The existing state is always worse than expected.

### Right-Sizing Checklist

Before finalizing engagement size, confirm:
- [ ] Divisions needed have been identified (not assumed to be all four)
- [ ] Compliance frameworks are known and factored into scope
- [ ] Greenfield vs brownfield is established
- [ ] Multi-region requirements are known
- [ ] Budget ceiling is established and engagement size fits within it
- [ ] Shared services (security, cost, docs, testing) effort is included in the estimate
