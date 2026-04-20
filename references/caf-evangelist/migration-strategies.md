## Migration Strategies — The 8 Rs

### Overview

Migration strategy selection determines the cost, risk, and timeline for each workload's
cloud journey. The 8 Rs provide a framework for making deliberate decisions about every
workload rather than defaulting to a single approach.

### The 8 Rs Decision Framework

```
What should we do with this workload?
├─ No longer needed?              → RETIRE (decommission)
├─ Can't move yet?                → RETAIN (keep on-prem, revisit later)
├─ Move as-is?                    → REHOST (lift-and-shift to IaaS VMs)
├─ Move to PaaS, minimal changes? → REPLATFORM (minor optimization)
├─ Optimize code for cloud?       → REFACTOR (code changes, same behavior)
├─ Need cloud-native design?      → REARCHITECT (significant redesign)
├─ Start completely over?         → REBUILD (new cloud-native from scratch)
└─ SaaS alternative exists?       → REPLACE (adopt existing SaaS product)
```

### Strategy Selection Guide

| Strategy | When to Use | Risk | Cost | Timeline |
|----------|------------|------|------|----------|
| **Retire** | Workload obsolete, no downstream dependencies | Low | Lowest | Immediate |
| **Retain** | Compliance blocker, not ready, or strategic hold | Low | None (status quo) | Deferred |
| **Rehost** | Stable workload, datacenter exit deadline | Low | Low | Weeks |
| **Replatform** | Want managed services, minimal code changes | Medium | Medium | Weeks–months |
| **Refactor** | Performance/cost optimization, reduce tech debt | Medium | Medium | Months |
| **Rearchitect** | Need microservices, event-driven, elastic scale | High | High | Months |
| **Rebuild** | Legacy beyond saving, greenfield opportunity | High | Highest | Months–quarters |
| **Replace** | SaaS solution exists, custom code unnecessary | Medium | Varies | Months |

### Assessment Criteria for Each Workload

Before selecting a strategy, evaluate each workload against:

- **Business criticality** — revenue impact, user count, regulatory exposure
- **Technical complexity** — dependencies, custom code, data gravity
- **Cloud readiness** — OS support, licensing, network requirements
- **Team capability** — skills available to execute the chosen strategy
- **Time pressure** — datacenter exit deadlines, contract expirations

### Azure Migrate for Assessment and Execution

Azure Migrate provides a centralized hub for migration activities:

- **Discovery and assessment** — inventory servers, databases, web apps
- **Dependency analysis** — agent-based or agentless dependency mapping
- **Migration execution** — replication, test migration, cutover orchestration
- **Database migration** — Azure Database Migration Service for SQL, MySQL, PostgreSQL
- **Web app migration** — App Service migration assistant

### Migration Waves

Organize migrations into waves rather than big-bang:

```
Wave 1: Low-risk rehost (build confidence, prove process)
Wave 2: Replatform candidates (introduce PaaS, demonstrate value)
Wave 3: Refactor / rearchitect (higher complexity, team has experience)
Wave 4: Replace / rebuild (strategic modernization)
Ongoing: Retire and retain decisions revisited each quarter
```

### Common Anti-Patterns

- **"Rehost everything first, modernize later"** — creates permanent technical debt.
  Organizations rarely go back to modernize. Plan modernization waves from the start.
- **Ignoring retire candidates** — migrating workloads nobody uses wastes budget.
- **No test migrations** — skipping test cutover leads to surprises during production migration.
- **Treating all workloads identically** — each workload deserves individual strategy assessment.
