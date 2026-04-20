## Engagement Model

Every engagement follows a five-phase lifecycle: Scope → Plan → Execute → Validate → Deliver. Phases are sequential — you do not skip phases, you do not reorder phases, and you do not start the next phase until the current phase is complete with its gates passed.

### Phase 1: Scope

**Purpose**: Understand what is being asked and determine what the engagement requires.

**Activities**:
- Gather requirements from stakeholders — what problem are we solving?
- Identify which divisions are needed — not every engagement requires all four
- Determine greenfield vs brownfield — this changes complexity, timeline, and risk
- Identify compliance frameworks — SOC2, HIPAA, PCI-DSS, FedRAMP each constrain architecture
- Establish budget constraints — monthly Azure spend ceiling and project labor budget
- Assess existing infrastructure — what already exists in Azure, on-prem, or other clouds
- Define success criteria — specific, measurable outcomes, not vague goals
- Estimate effort — small, medium, or large engagement (see engagement-sizing.md)

**Output**: Scoping document with requirements, divisions involved, effort estimate, timeline, and success criteria.

### Phase 2: Plan

**Purpose**: Assign the right specialists, define milestones, and map dependencies.

**Activities**:
- Division leads assess the problem from their perspective
- Assign specific specialists to workstreams based on required expertise
- Define milestones with clear deliverables and dates
- Map dependency chains — identity before platform, platform before services, architecture before IaC
- Identify parallelization opportunities — what workstreams can run concurrently
- Schedule gate checkpoints — security, cost, and documentation reviews at phase boundaries
- Establish communication cadence with stakeholders

**Output**: Engagement plan with workstream assignments, milestone schedule, dependency map, and gate checkpoints.

### Phase 3: Execute

**Purpose**: Specialists do the technical work with structured handoffs between divisions.

**Activities**:
- Identity & Productivity establishes the identity foundation first — Entra ID, Conditional Access, managed identity strategy
- Platform Engineering deploys landing zones, networking foundation, governance policies
- Azure Specialty configures services — compute, database, storage, networking, monitoring, integration
- DevOps builds CI/CD pipelines and IaC modules once architecture decisions are made
- Handoffs between divisions follow a clear protocol: defined deliverable, acceptance criteria, documentation of decisions
- Shared services participate throughout — security reviews architecture, cost tracks spend, documentation captures decisions

**Gate**: Security sign-off on architecture and identity model before proceeding to production workloads.
**Gate**: Cost approval on actual vs projected spend.

### Phase 4: Validate

**Purpose**: Confirm everything works and meets requirements before production.

**Activities**:
- Testing & Validation engineer validates deployed infrastructure against requirements
- Security conducts compliance validation, Defender for Cloud posture check, penetration testing
- Cost reviews actual spend, identifies right-sizing opportunities, plans commitment discounts
- Monitoring engineer verifies alerting, dashboards, and runbooks are operational
- Division leads confirm their workstreams meet acceptance criteria

**Gate**: Security sign-off for production readiness.
**Gate**: Cost approval for ongoing operational budget.

### Phase 5: Deliver

**Purpose**: Hand off to the customer with complete documentation and knowledge transfer.

**Activities**:
- Documentation writer completes architecture docs, operational runbooks, decision records, deployment procedures
- Training delivered to customer operations team — how to operate, monitor, troubleshoot, and scale
- Automation engineer deploys and tests operational automation scripts and runbooks
- Retrospective agent conducts post-engagement review — what worked, what didn't, lessons learned
- Engagement formally closed only after all documentation is complete

**Gate**: Documentation must be complete before close. No exceptions.
