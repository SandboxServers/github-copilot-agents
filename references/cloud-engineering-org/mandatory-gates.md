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
