## Cloud Adoption Anti-Patterns

### Purpose
Broad anti-patterns observed across cloud adoption programs — not CAF-specific
(see [caf-anti-patterns.md](caf-anti-patterns.md) for CAF phase-specific mistakes).
These patterns apply regardless of framework and represent organizational, strategic,
and execution failures that derail cloud programs.

### Skipping the Strategy Phase
Jumping directly to migration without business alignment.
- No defined business outcomes = no way to measure success
- Cloud becomes a technology project instead of a business transformation
- Executive sponsorship evaporates when ROI questions can't be answered
- Fix: spend 2–4 weeks defining motivations, outcomes, and first adoption project
- Reference: [CAF Strategy](https://learn.microsoft.com/azure/cloud-adoption-framework/strategy/)

### Lift-and-Shift Everything Without Modernization Assessment
Treating every workload as a VM rehost candidate.
- Ignores the 8 Rs of rationalization (rehost, replatform, refactor, rearchitect, rebuild, replace, retire, retain)
- Runs the same inefficient architecture at higher cost
- Creates "cloud is more expensive than on-prem" narrative
- Always assess: some workloads should be replaced (SaaS) or retired entirely
- PaaS savings of 30–60% are common for workloads that can replatform

### No Landing Zone: Deploying Directly to Production Subscriptions
Workloads deployed into unstructured subscriptions with ad-hoc configuration.
- No management group hierarchy, no consistent RBAC, no shared networking
- Shadow subscriptions proliferate — each team builds their own "foundation"
- Retrofitting governance around live production workloads is 5–10x harder
- Deploy at minimum an MVP landing zone before first production workload
- Reference: [ALZ architecture](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/landing-zone/)

### Governance as Afterthought
Adding policies, cost controls, and security baselines after resources already exist.
- Existing non-compliant resources create immediate remediation backlog
- Teams resist policy enforcement because it breaks their running workloads
- Cost overruns discovered months after they started accumulating
- Start governance in audit mode on day one, progress to deny as teams mature
- Minimum: cost alerts, required tags (audit), allowed regions, no broad Owner roles

### Single Team Owns Everything
No cloud center of excellence — one team is bottleneck for all cloud decisions.
- Central IT becomes gatekeeper, creating 2–4 week wait times for simple requests
- Teams bypass governance to avoid the bottleneck — shadow IT emerges
- Knowledge stays siloed in one team instead of distributed across the organization
- Establish a CCoE model: central team sets guardrails, application teams self-serve
- Reference: [CCoE functions](https://learn.microsoft.com/azure/cloud-adoption-framework/organize/cloud-center-of-excellence)

### Not Securing Executive Sponsorship
Cloud adoption driven bottom-up without C-level commitment.
- Budget fights at every phase — no pre-approved funding model
- Competing organizational priorities override cloud program
- No authority to enforce governance or operating model changes
- Cloud adoption is organizational transformation — requires executive air cover
- Identify a sponsor who can remove cross-team blockers and secure funding

### Ignoring Organizational Change Management
Treating cloud adoption as purely technical with no people strategy.
- Teams not trained on new tools, processes, or operating models
- Resistance from staff who see cloud as a threat to their roles
- No communication plan = rumors and anxiety replace clarity
- Address the human side: skills plans, role mapping, regular communication
- Reference: [Organizational alignment](https://learn.microsoft.com/azure/cloud-adoption-framework/organize/)

### Over-Planning: 12-Month Waterfall Plans for Cloud Adoption
Building exhaustive multi-year plans before deploying anything.
- Cloud changes faster than any waterfall plan can accommodate
- Analysis paralysis — teams plan for 6 months, technology shifts underneath them
- No feedback loop from actual migration experience
- Plan in 90-day increments. First wave teaches more than 6 months of planning
- Use the Plan phase for digital estate assessment, not detailed project scheduling

### No Skills Assessment
Assuming existing team can handle cloud without training investment.
- On-prem networking skills don't translate directly to cloud networking
- Security teams unfamiliar with identity-based perimeters and Zero Trust
- Developers don't know PaaS patterns — they build VM-based solutions by default
- Run a skills gap analysis early. Map training to adoption timeline
- Microsoft Learn paths, certifications, and hands-on labs are free starting points

### Shadow IT: Teams Adopting Cloud Outside Governance
Business units spinning up cloud resources without central visibility.
- Credit card subscriptions with no cost management or security baseline
- Data sovereignty violations when resources land in unapproved regions
- No disaster recovery, no backup, no incident response for shadow workloads
- Don't just block — provide a fast, governed path that's easier than going rogue
- Subscription vending with guardrails makes the right way the easy way

### Cost Estimation Without Proof-of-Concept Validation
Building business cases from TCO calculator alone without real-world validation.
- TCO calculator underestimates operational costs (monitoring, patching, training)
- Doesn't account for right-sizing — estimates based on current on-prem sizing
- Reserved instance and savings plan discounts assumed but not committed
- Always run a PoC for representative workloads before committing to business case
- Validate Azure Hybrid Benefit eligibility and licensing implications

### Not Establishing Cloud Operating Model Before Migration
Migrating workloads without deciding who operates them post-migration.
- Day-2 operations undefined: who patches? who monitors? who responds to incidents?
- Teams argue over responsibility boundaries after workloads are live
- Operational debt accumulates because nobody owns the ongoing work
- Choose operating model (centralized, decentralized, enterprise, distributed) before wave 1
- Reference: [Operating models](https://learn.microsoft.com/azure/cloud-adoption-framework/operating-model/)
