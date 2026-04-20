# Disaster Recovery Patterns

Strategies for business continuity across Azure region failures.

> **Source:** Azure Well-Architected Framework reliability pillar, Microsoft Learn DR guidance.

## Key Concepts

| Term | Definition |
|------|-----------|
| **RTO** (Recovery Time Objective) | Maximum acceptable downtime |
| **RPO** (Recovery Point Objective) | Maximum acceptable data loss (time) |
| **Failover** | Switching traffic from primary to secondary region |
| **Failback** | Returning traffic to primary after recovery |

## Pattern Comparison

| Pattern | RTO | RPO | Cost | Complexity |
|---------|-----|-----|------|------------|
| Active-Active | Near zero | Near zero | High (2x resources) | High |
| Active-Passive (Hot Standby) | Minutes | Minutes | Medium-High | Medium |
| Active-Passive (Cold Standby) | Hours | Minutes-Hours | Medium | Medium |
| Pilot Light | Hours | Minutes | Low-Medium | Low-Medium |
| Backup & Restore | Hours-Days | Hours | Low | Low |

## Active-Active

Both regions serve live traffic simultaneously. Azure Front Door or Traffic Manager distributes requests globally.

**Architecture:**
- Identical compute stacks in two+ regions
- Cosmos DB with multi-region writes or Azure SQL with active geo-replication
- Azure Front Door for global routing with health probes per region
- Shared-nothing design — each region is fully self-contained

**When to use:** Mission-critical workloads with near-zero RTO/RPO requirements. E-commerce, financial services, global SaaS products.

**Gotchas:** Data conflicts with multi-region writes. Session affinity breaks if a region fails. Costs are ~2x. Testing failover is complex.

## Active-Passive (Hot Standby)

Primary region serves all traffic. Secondary region is deployed and running but receives no user traffic until failover.

**Architecture:**
- Full compute stack in secondary region, scaled down but running
- Azure SQL geo-replication (async) or Cosmos DB single-write-region with read replicas
- Front Door or Traffic Manager with failover routing
- DNS TTL set low enough for timely failover (~60 seconds)

**When to use:** Business-critical workloads that can tolerate minutes of downtime. Most production SaaS applications.

**Gotchas:** Secondary region may need scale-up during failover. Async replication means some data loss (RPO > 0). Test failover regularly — untested DR is not DR.

## Pilot Light

Minimal infrastructure in the secondary region — only data replication and core services. Compute is provisioned on-demand during failover.

**Architecture:**
- Database replication to secondary region (always running)
- IaC templates ready to deploy compute stack (Bicep/Terraform)
- DNS-based failover via Traffic Manager
- Compute provisioned during failover event (10-60 min depending on complexity)

**When to use:** Cost-sensitive workloads with RTO tolerance of 1-4 hours. Internal applications, batch processing systems.

**Gotchas:** Provisioning compute during an incident adds stress. Region capacity may be constrained during widespread outages. Pre-reserve capacity with on-demand capacity reservations if RTO is critical.

## Backup & Restore

No secondary region infrastructure. Recovery relies on restoring from backups to a new region.

**When to use:** Dev/test environments, non-critical internal tools, workloads with RTO tolerance of 8+ hours.

**Implementation:** Azure Backup with cross-region restore. Geo-redundant storage (GRS) for blobs. Point-in-time restore for databases. IaC to rebuild infrastructure in target region.

## Implementation Checklist

| Step | Action |
|------|--------|
| 1 | Define RTO and RPO for each workload tier |
| 2 | Select DR pattern based on RTO/RPO/budget |
| 3 | Choose paired or non-paired secondary region |
| 4 | Implement data replication (SQL geo-rep, Cosmos multi-region, GRS) |
| 5 | Configure global load balancer (Front Door or Traffic Manager) |
| 6 | Automate failover with IaC — no manual portal steps |
| 7 | Test failover quarterly — DR that isn't tested doesn't work |
| 8 | Document runbooks for failover and failback procedures |
| 9 | Set up alerts for replication lag and region health |
