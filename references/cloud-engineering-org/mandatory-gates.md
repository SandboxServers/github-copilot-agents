## Mandatory Gates

Every engagement passes through four non-negotiable gates. These are not optional checkpoints — they are hard requirements that block progress until satisfied. No engagement ships without passing all four.

### Gate 1: Security Review (Veto Power)

**Authority**: @security-compliance-analyst has veto power. If security says no, the answer is no.

**When applied**: Every phase boundary, every architecture decision, every production deployment.

**What is reviewed**:
- Identity model — least privilege, managed identity usage, RBAC design
- Network security — private endpoints, NSGs, firewall rules, DNS resolution
- Data protection — encryption at rest and in transit, key management, data residency
- Compliance controls — framework-specific requirements, audit logging, retention policies
- Threat protection — Defender coverage, security baselines, vulnerability management

**Blocking conditions**: Security can block any deployment at any phase. If the security review identifies a gap, the design changes or the timeline shifts. "We'll fix it later" is not an acceptable response to a security finding.

### Gate 2: Cost Review (Budget Approval)

**Authority**: @cost-optimization-specialist must approve all new Azure spend before provisioning.

**When applied**: Before any Azure resources are provisioned, at each phase boundary, before commitment purchases.

**What is reviewed**:
- Projected vs actual spend — are we tracking to budget?
- Right-sizing opportunities — are resources appropriately sized for workload?
- Commitment discount eligibility — reservations, savings plans, hybrid benefit
- Egress and cross-region costs — often overlooked until the bill arrives
- Cost allocation and tagging — can spend be attributed to workloads and teams?
- Long-term sustainability — can the customer afford to run this operationally?

**Blocking conditions**: If cost says the budget doesn't work, the design changes. The organization does not approve "we'll optimize later" — cost is optimized now or the architecture is redesigned.

### Gate 3: Documentation Deliverables (Close Gate)

**Authority**: @documentation-writer controls the close gate. Engagement cannot close without completed documentation.

**When applied**: Before any engagement can be marked complete.

**What is required**:
- Architecture decision records — what was decided and why, with alternatives considered
- Operational runbooks — how to operate, monitor, troubleshoot, and recover
- Deployment documentation — how to deploy, rollback, and scale
- Security documentation — compliance controls, access model, incident response procedures
- Cost documentation — budget, projections, optimization plan
- Training materials — for the customer's operations team

**Blocking conditions**: If documentation is not complete, the engagement is not complete. "We'll document it later" means the documentation never gets written.

### Gate 4: Testing Validation (Production Readiness)

**Authority**: @testing-validation-engineer validates before production deployment.

**When applied**: Before any workload receives production traffic.

**What is validated**:
- Infrastructure deployment matches architecture specifications
- Connectivity and network paths function as designed
- Security controls are in place and functioning — not just configured but verified
- Monitoring and alerting are operational — alerts fire, dashboards populate, runbooks work
- Disaster recovery and failover procedures are tested, not just documented
- Performance meets defined requirements under expected load

**Blocking conditions**: If testing finds a gap, the deployment does not proceed to production. Issues are fixed and retested before production cutover.
## The Three Mandatory Gates

### Gate 1: Security (VETO POWER)

**When**: Every phase boundary, every architecture decision, every production deployment.

**What security reviews**:
- Identity model (least privilege, managed identity usage, RBAC design)
- Network security (private endpoints, NSGs, firewall rules, DNS)
- Data protection (encryption at rest/transit, key management, data residency)
- Compliance (framework-specific controls, audit logging, retention)
- Threat protection (Defender coverage, security baselines, vulnerability management)

**Authority**: If security says no, the answer is no. The design changes, the timeline shifts, or the approach pivots. Security is not a suggestion.

### Gate 2: Cost (BUDGET APPROVAL)

**When**: Before any Azure resources are provisioned, at each phase boundary, before commitment purchases.

**What cost reviews**:
- Projected vs actual spend
- Right-sizing opportunities
- Commitment discount eligibility (reservations, savings plans)
- Egress and cross-region cost exposure
- Cost allocation and tagging strategy
- Operational cost sustainability (can the customer afford to run this long-term?)

**Authority**: If cost says the budget doesn't work, the design changes. The Org Lead does not approve "we'll optimize later" — cost is optimized now or the architecture is redesigned.

### Gate 3: Documentation (CLOSE GATE)

**When**: Before any engagement can be marked complete.

**What documentation covers**:
- Architecture decision records (what was decided and why)
- Operational runbooks (how to operate, monitor, troubleshoot)
- Deployment documentation (how to deploy, rollback, scale)
- Security documentation (compliance controls, access model, incident response)
- Cost documentation (budget, projections, optimization plan)
- Training materials (for the customer's operations team)

**Authority**: If docs aren't done, the engagement isn't done. The Org Lead does not close engagements with "we'll document it later" — that documentation never gets written.
