## Cross-Division Coordination

### Dependency Flow

Work flows through divisions in a predictable sequence. Understanding these dependencies prevents bottlenecks and rework.

```
Identity & Productivity (must go first)
  → Entra ID foundation, Conditional Access, managed identity strategy
  → Gates: Platform, Specialty, DevOps all depend on identity decisions

Platform Engineering (second)
  → Landing zones, subscriptions, networking foundation, governance
  → Gates: Specialty and DevOps need infrastructure before deploying services

Azure Specialty (third, often parallelized)
  → Network topology, compute, database, storage, integration, monitoring
  → Gates: DevOps needs architecture decisions before writing IaC

DevOps (builds on decisions from Specialty and Platform)
  → CI/CD pipelines, IaC modules, deployment automation
  → Gates: Requires compute targets, architecture patterns, and environments

Shared Services (throughout)
  → Security, Cost, Documentation, Testing, Automation, Retrospective
  → Participate at every phase, not just at the end
```

### Common Cross-Division Conflicts

**Network topology affects compute placement**: Network engineer designs hub-spoke with specific IP ranges and private endpoint placement. Compute engineer needs subnet sizing that accommodates AKS node pools or VMSS. If these decisions are made independently, one team's design constrains the other. Resolution: network and compute engineers coordinate during Phase 3, not after.

**Security requirements increase cost**: Security mandates private endpoints, WAF, DDoS protection, Defender for Cloud plans, and premium firewall SKUs. Cost analyst sees the bill growing beyond budget. Resolution: Security and Cost negotiate — identify which controls are compliance-mandated vs best-practice, and find cost-effective alternatives for non-mandatory controls.

**Monitoring adds complexity**: Monitoring engineer wants agents on every resource, diagnostic settings everywhere, Log Analytics ingesting from all sources. This increases compute overhead and storage costs. Resolution: monitoring scope is defined during planning, not discovered during implementation.

**IaC can't start until architecture is decided**: DevOps wants to start writing Terraform early. Azure Specialty hasn't finalized whether to use AKS or App Service, SQL MI or Azure SQL. Resolution: architecture decisions are documented as they're made, and DevOps begins IaC for decided components rather than waiting for the full architecture.

**Identity decisions delay everything**: Entra ID specialist hasn't finalized the Conditional Access policy set or managed identity strategy. Platform and Specialty can't assign RBAC or configure service authentication. Resolution: Identity decisions are Phase 1 priority — this division starts first for a reason.

### Conflict Resolution

1. **Within a division**: The division lead resolves conflicts between their specialists
2. **Between two divisions**: Division leads negotiate directly, seeking a compromise that satisfies both constraints
3. **Unresolvable between leads**: Escalate to the Org Lead, who makes the final call based on engagement priorities
4. **Mandatory gate conflict**: The gate authority (Security, Cost, Documentation) has final say within their domain — the Org Lead adjusts the plan, not the gate requirement

### Handoff Protocol

When work passes from one division to another:
1. **Define the deliverable** — what exactly is being handed off and in what state
2. **State acceptance criteria** — what the receiving division needs to proceed
3. **Document decisions** — what was decided, what alternatives were considered, what constraints were discovered
4. **Name a contact point** — who to ask when questions arise during the next phase
