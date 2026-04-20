## CAF Best Practices

### Purpose
Proven practices for successful cloud adoption programs using the Cloud Adoption
Framework. These are distilled from Microsoft guidance and real-world adoption
experience.

### Start with Strategy: Business Outcomes First
Define why before how — technology decisions follow business motivations.
- Identify business motivations: cost reduction, agility, innovation, compliance
- Define measurable outcomes: "reduce time-to-market by 40%" not "move to cloud"
- Align stakeholders early — executive sponsor, finance, security, operations
- Build a business case with cloud economics (not just TCO — include innovation value)
- Strategy phase should take 2–4 weeks, not 2–4 months
- Reference: [CAF Strategy](https://learn.microsoft.com/azure/cloud-adoption-framework/strategy/)

### Landing Zone First: Foundation Before Workloads
Deploy infrastructure foundation before any production workload.
- Use ALZ reference architecture as starting point — customize, don't build from scratch
- Minimum viable landing zone: management groups, hub network, baseline policies, identity
- Full ALZ includes Defender for Cloud, Log Analytics, and policy-driven governance
- Landing zone is not a one-time deployment — it evolves with the organization
- Consider ALZ Bicep or Terraform accelerators for repeatable deployment
- Reference: [Landing zone design areas](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/landing-zone/design-areas)

### Wave-Based Migration: Prioritize by Value and Complexity
Organize migration into waves, not a big bang.
- Wave 1: low-risk, well-understood workloads (build confidence, refine processes)
- Wave 2: moderate complexity (apply lessons learned from wave 1)
- Wave 3+: business-critical and complex workloads (team is experienced by now)
- Each wave should include assessment, migration, optimization, and handoff to operations
- 10–20 workloads per wave is a typical cadence for mid-size organizations
- Use Azure Migrate for assessment but supplement with application team knowledge

### Governance from Day 1: Start with Audit, Progress to Deny
Governance is not a phase — it runs continuously from the beginning.
- Deploy governance MVP before first workload: cost alerts, tagging (audit), allowed regions
- Use Azure Policy in audit mode first — visibility before enforcement
- Transition to deny effect after teams understand the guardrails (30–60 day ramp)
- Five disciplines: cost management, security baseline, identity baseline, resource consistency, deployment acceleration
- Assign governance team or CCoE function — governance without ownership fails
- Reference: [CAF Govern](https://learn.microsoft.com/azure/cloud-adoption-framework/govern/)

### Skills Enablement: Training Aligned to Adoption Timeline
Invest in people before expecting them to deliver cloud outcomes.
- Run a skills gap analysis during the Plan phase — map current vs. required capabilities
- Align training to migration waves: network skills before wave 1, PaaS skills before wave 3
- Use Microsoft Learn paths and certifications (AZ-900, AZ-104, AZ-305) as baselines
- Hands-on labs and sandbox environments accelerate learning more than classroom training
- Budget for ongoing skills development — cloud evolves continuously

### Cloud Economics: Validate TCO with Proof-of-Concept
Business cases built on spreadsheets alone are unreliable.
- Run representative workloads in Azure for 30–60 days to gather real cost data
- Include operational costs: monitoring, security tooling, staffing, training
- Factor in Azure Hybrid Benefit and reserved instances for accurate comparison
- Revisit cost models quarterly — initial estimates are always wrong
- Use Azure Cost Management + Advisor recommendations from PoC data
- Reference: [Cloud economics](https://learn.microsoft.com/azure/cloud-adoption-framework/strategy/cloud-migration-business-case)

### Operating Model: Choose Based on Organizational Maturity
The operating model must match the organization's actual capability, not aspirations.
- Centralized: single IT team manages all cloud resources (works for small orgs)
- Decentralized: each business unit manages their own cloud (requires strong governance)
- Enterprise/CCoE hybrid: central team sets guardrails, application teams self-serve
- Distributed: fully autonomous teams with shared platform (requires highest maturity)
- Assess honestly — choosing a model beyond organizational capability creates friction
- Reference: [Operating model comparison](https://learn.microsoft.com/azure/cloud-adoption-framework/operating-model/compare)

### Security Baseline: Implement Before First Workload
Security is not a gate at the end — it's a foundation at the beginning.
- Enable Microsoft Defender for Cloud before first workload deployment
- Implement Azure landing zone security baseline (NSGs, Azure Firewall, DDoS protection)
- Zero Trust principles: verify explicitly, least privilege, assume breach
- Security policies in audit mode initially — same progression as governance
- Integrate security team into cloud adoption from Strategy phase forward
- Reference: [CAF Security](https://learn.microsoft.com/azure/cloud-adoption-framework/secure/)

### Continuous Improvement: Measure, Learn, Iterate
Cloud adoption is never "done" — mature organizations optimize continuously.
- Establish KPIs during Strategy phase and measure against them regularly
- Post-migration optimization: right-sizing, reserved instances, architecture review
- Retrospectives after each migration wave — what worked, what didn't, what to change
- Governance and security posture should improve with each iteration
- Cloud Adoption Framework itself evolves — revisit guidance quarterly
- Well-Architected Framework reviews for critical workloads post-migration
- Reference: [CAF Manage](https://learn.microsoft.com/azure/cloud-adoption-framework/manage/)
