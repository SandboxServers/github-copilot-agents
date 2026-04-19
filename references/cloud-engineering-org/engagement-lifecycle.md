## Engagement Lifecycle

### Phase 0: Scoping & Discovery

Before any technical work begins, the Org Lead asks the questions that scope the engagement:

**Mandatory scoping questions:**
1. **Greenfield or brownfield?** — Starting fresh vs evolving existing. This changes everything — timeline, complexity, migration needs, risk.
2. **Timeline?** — Weeks, months, quarters? This determines how much can be parallelized vs what must be serialized.
3. **Stakeholders?** — Who has decision authority? Who has veto power? Who are the end users? Who signs off on completion?
4. **Success criteria?** — What does done look like? Be specific. "Migrate to Azure" is not a success criterion. "All production workloads running in Azure with <100ms latency and 99.9% uptime" is.
5. **Budget?** — What's the monthly Azure spend ceiling? What's the project budget for labor? Are there commitment discounts already in place?
6. **Compliance frameworks?** — SOC2, HIPAA, PCI-DSS, FedRAMP, GDPR? Each one constrains architecture, data residency, encryption, access controls.
7. **Team capabilities?** — What does the customer's team know today? What skills need to be built? Who will operate this after handoff?
8. **Existing infrastructure?** — What's already in Azure? What's on-prem? What's in other clouds? What integrations exist?

### Phase 1: Assessment & Planning

**Division involvement**: All four leads assess the problem from their perspective.

| Division | Assessment Focus |
|----------|-----------------|
| Platform Engineering | Landing zone readiness, governance gaps, subscription structure, CAF alignment |
| Identity & Productivity | Identity foundation, M365 integration points, zero trust readiness, licensing gaps |
| Azure Specialty | Technology selection, architecture patterns, domain-specific requirements |
| DevOps | Deployment pipeline needs, current CI/CD state, IaC readiness, developer experience |

**Shared services involvement**:
- Security reviews the compliance requirements and establishes security baselines
- Cost models the projected Azure spend and identifies optimization opportunities
- Documentation creates the engagement plan document

**Output**: Engagement plan with phases, milestones, division assignments, dependencies, and gate checkpoints.

### Phase 2: Foundation

**Sequencing**: Identity → Platform → (then everything else can begin)

1. **Identity & Productivity**: Establish Entra ID foundation, Conditional Access baseline, device compliance policies. Identity gates everything else — Azure resources need RBAC, pipelines need service principals, M365 services need user identities.
2. **Platform Engineering**: Deploy landing zones, management group hierarchy, subscription vending, networking foundation, governance policies. Infrastructure needs a home before it can be built.

**Gate**: Security sign-off on identity model and landing zone architecture before proceeding.
**Gate**: Cost approval on projected infrastructure spend.

### Phase 3: Build

**Sequencing**: Can often parallelize across divisions, with dependency management.

| Workstream | Division | Dependencies |
|-----------|----------|-------------|
| Network topology | Azure Specialty (network engineer) | Landing zones from Phase 2 |
| Database architecture | Azure Specialty (database specialist) | Network topology |
| Compute platforms | Azure Specialty (compute engineer) | Network, database endpoints |
| Integration patterns | Azure Specialty (integration architect) | Compute, database |
| AI/ML workloads | Azure Specialty (AI specialist) | Compute, storage, network |
| Data pipelines | Azure Specialty (data engineer) | Storage, compute, database |
| Monitoring | Azure Specialty (monitoring engineer) | All resources (wired up as deployed, not after) |
| CI/CD pipelines | DevOps | Compute targets, IaC modules |
| IaC modules | DevOps + Platform | Architecture decisions from Specialty |
| M365 configuration | Identity & Productivity | Identity foundation from Phase 2 |
| Operational automation | Shared Services (PowerShell) | Infrastructure from Specialty |

**Gate**: Security review of architecture and deployed resources before production traffic.
**Gate**: Cost review of actual vs projected spend.

### Phase 4: Validate & Harden

1. **Testing & Validation**: Validate deployed infrastructure against requirements
2. **Security**: Penetration testing, compliance validation, Defender for Cloud posture check
3. **Cost**: Actual spend review, right-sizing recommendations, commitment discount planning
4. **Monitoring**: Verify alerting, dashboards, runbooks are operational

**Gate**: Security sign-off for production readiness.
**Gate**: Cost approval for ongoing operational budget.

### Phase 5: Handoff & Close

1. **Documentation**: Architecture docs, runbooks, decision records, operational procedures — all complete
2. **Training**: Customer team trained on operations, monitoring, incident response
3. **Automation**: Operational automation deployed and tested
4. **Retrospective**: Post-engagement review — what worked, what didn't, lessons learned, action items

**Gate**: Documentation must be complete before close. No exceptions.
