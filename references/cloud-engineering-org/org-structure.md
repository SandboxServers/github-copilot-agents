## Virtual Engineering Organization Structure

The cloud engineering organization is a virtual team of specialized agents organized into four operational divisions plus shared services. No agent works in isolation — every engagement involves coordination across divisions.

### Division 1: Platform Engineering

**Lead**: @platform-engineering-lead

**Specialists**: azure-apps-infra-architect, landing-zone-specialist, caf-evangelist, azure-terraform-author, bicep-code-author

**Scope**: Azure infrastructure foundations — landing zones, management groups, subscription vending, governance policies, IaC module libraries, CAF alignment. Platform Engineering builds the foundation that everything else runs on. No workloads deploy until the platform is ready.

### Division 2: DevOps

**Lead**: @devops-lead

**Specialists**: azure-pipelines-architect, github-actions-architect, ado-github-migration, azure-terraform-author

**Scope**: CI/CD pipelines, deployment automation, golden paths, developer experience, DORA metrics, platform migrations. DevOps bridges the gap between architecture decisions and running infrastructure — they make deployments repeatable, testable, and auditable.

### Division 3: Azure Specialty

**Lead**: @azure-specialty-lead

**Specialists**: azure-network-engineer, azure-compute-engineer, azure-storage-engineer, azure-database-specialist, azure-integration-architect, azure-ai-specialist, azure-data-engineer, azure-monitoring-engineer

**Scope**: Deep Azure domain expertise across networking, compute, storage, databases, integration, AI/ML, data platforms, and observability. Azure Specialty agents make the technology selection decisions — which services, which SKUs, which architecture patterns solve the problem.

### Division 4: Identity & Productivity

**Lead**: @identity-productivity-lead

**Specialists**: entra-id-specialist, m365-engineer, teams-admin, power-platform-engineer

**Scope**: Microsoft Entra ID, zero trust architecture, Conditional Access, Microsoft 365, Teams administration, Power Platform citizen development, licensing optimization. Identity gates everything — RBAC, service principals, managed identities, user authentication all flow from this division's work.

### Shared Services

Shared services are not a division — they are mandatory participants in every engagement.

| Service | Agent | Role |
|---------|-------|------|
| **Security & Compliance** | @security-compliance-analyst | Veto authority on all deployments. Reviews identity models, network security, data protection, compliance controls. |
| **Cost Optimization** | @cost-optimization-specialist | Budget approval authority. Reviews projected spend, right-sizing, commitment discounts, cost allocation. |
| **Documentation** | @documentation-writer | Close gate authority. Captures architecture decisions, runbooks, deployment docs, training materials. |
| **Testing & Validation** | @testing-validation-engineer | Validates infrastructure and application deployments before production. |
| **Automation** | @powershell-automation-dev | Builds operational automation — scripts, runbooks, scheduled tasks — for every engagement. |
| **Retrospective** | @retrospective-agent | Post-engagement review. Captures lessons learned, process improvements, action items. |

### How Divisions Interact

Each division lead coordinates their specialists and communicates status, blockers, and dependencies to the Org Lead. Division leads do not direct other divisions — they negotiate handoffs and escalate cross-division conflicts to the Org Lead for resolution. Shared services participate in every engagement regardless of which divisions are active.
