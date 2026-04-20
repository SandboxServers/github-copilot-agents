## CAF Gotchas

### Purpose
Non-obvious pitfalls that catch teams even when they're following CAF guidance.
These aren't anti-patterns (deliberate mistakes) — they're surprises that emerge
from reasonable decisions.

### CAF Phases Are Not Linear
Governance and security are continuous, not sequential.
- Common misconception: finish Strategy → Plan → Ready → Migrate in order
- Reality: Govern and Manage run in parallel from day one and never stop
- Security baseline must be in place before first workload, not after Migrate
- Teams that treat CAF as a waterfall sequence skip governance until it's too late
- Revisit Strategy as business objectives evolve — it's not a one-time exercise

### Landing Zone Complexity
ALZ accelerator creates 100+ resources — teams underestimate the blast radius.
- Full ALZ deployment includes management groups, policies, Log Analytics, Defender, networking, and identity resources
- Teams expecting a "simple foundation" are overwhelmed by the resource count
- Start with the platform landing zone conceptual architecture, then decide scope
- Consider ALZ Bicep or Terraform modules for incremental deployment
- Small organizations may not need full ALZ — an MVP landing zone can suffice
- Reference: [ALZ accelerator](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/landing-zone/implementation-options)

### Migration Assessment Tool Limitations
Azure Migrate has blind spots for certain workload types.
- Oracle, SAP, and mainframe workloads need specialized assessment tools
- Dependency mapping requires agents or agentless appliance — neither captures everything
- Performance data needs 30+ days of collection for reliable right-sizing
- Application-level dependencies (API calls, shared databases) aren't detected automatically
- Supplement Azure Migrate with application team interviews and network flow analysis

### TCO Calculator Often Underestimates Operational Costs
The Azure TCO calculator is a starting point, not a business case.
- Doesn't include staffing costs for new skills (cloud architects, SREs)
- Ignores training investment needed for operations and development teams
- Networking egress costs are frequently underestimated for data-heavy workloads
- Monitoring, security tooling, and backup licensing add 15–25% to raw compute costs
- Always validate with a proof-of-concept running real workloads at realistic scale

### Cloud Operating Model: Wrong Choice Is Hard to Reverse
Choosing centralized vs. decentralized early becomes deeply embedded.
- Centralized model creates bottlenecks as the organization scales past 20 workloads
- Fully decentralized model leads to inconsistent governance and duplicated effort
- Hybrid/CCoE model is usually right but requires executive sponsorship to establish
- Changing operating models mid-migration disrupts teams and slows adoption
- Assess organizational maturity honestly — aspirational models fail without matching capability
- Reference: [Operating model comparison](https://learn.microsoft.com/azure/cloud-adoption-framework/operating-model/compare)

### Subscription Vending: Too Many Subscriptions = Management Overhead
Subscription-per-workload sounds clean but creates sprawl.
- Each subscription needs cost management, RBAC, networking, and monitoring configuration
- Azure has subscription-level limits (e.g., ExpressRoute circuits, VNet peerings)
- Policy assignment at scale requires management group hierarchy — not per-subscription
- Balance isolation benefits against operational overhead
- Automate subscription vending with Infrastructure as Code — manual creation doesn't scale

### Policy Enforcement: Too Aggressive Early
Starting with deny-effect policies before teams understand the guardrails.
- Developers can't deploy anything → they open support tickets or bypass governance
- Shadow IT increases because the governed path is harder than the ungoverned path
- Audit-mode policies generate compliance reports without blocking deployments
- Start with audit, communicate findings, then transition to deny after 30–60 days
- Always provide an exception/exemption process — absolute deny with no escape valve creates friction
- Reference: [Azure Policy effects](https://learn.microsoft.com/azure/governance/policy/concepts/effects)
