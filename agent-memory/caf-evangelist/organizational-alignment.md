## Organizational Alignment

### Cloud Operating Model

The operating model defines how the organization delivers and manages cloud services.
Three primary models exist — most organizations land somewhere between them.

| Model | Description | Best For |
|-------|-----------|----------|
| **Centralized** | Central IT owns all cloud operations | Organizations with strong central IT, regulated industries |
| **Federated** | App teams operate independently with shared standards | Large orgs with autonomous business units |
| **CCoE (Cloud Center of Excellence)** | Cross-functional team enables self-service within guardrails | Organizations pursuing platform engineering |

### RACI for Cloud Operations

Define clear ownership across cloud responsibilities:

| Responsibility | Cloud Strategy | Platform Team | App Teams | Security | FinOps |
|---------------|---------------|--------------|-----------|----------|--------|
| Business case / ROI | **A/R** | C | I | I | C |
| Landing zone design | C | **A/R** | I | C | I |
| Workload deployment | I | C | **A/R** | C | I |
| Security policy | C | C | I | **A/R** | I |
| Cost management | I | C | R | I | **A/R** |
| Incident response | I | R | R | **A/R** | I |

*A = Accountable, R = Responsible, C = Consulted, I = Informed*

### CCoE vs Centralized vs Federated

**CCoE characteristics:**
- Cross-functional (platform, security, FinOps, app dev representation)
- Builds guardrails, not gates — enables self-service
- Maintains reference architectures and approved patterns
- Requires executive sponsorship and cultural shift
- Maximum innovation speed with minimum governance overhead

**Centralized characteristics:**
- Central IT makes all cloud decisions and manages all resources
- Tight control, slower delivery, potential bottleneck
- Works when compliance requirements demand centralized oversight

**Federated characteristics:**
- Each business unit manages its own cloud environment
- Fast for individual teams, risk of duplication and inconsistency
- Requires strong shared standards to avoid drift

### Skills Gap Analysis

Assess current team capabilities against required cloud roles:

- **Cloud Architect** — solution design, Well-Architected Framework
- **Platform Engineer** — landing zones, IaC, CI/CD, policy management
- **Security Engineer** — Defender, Sentinel, identity, compliance
- **FinOps Practitioner** — cost optimization, budgeting, chargeback
- **Site Reliability Engineer** — monitoring, incident response, automation

### Training Paths

- **Microsoft Learn** — free, role-based learning paths for all Azure roles
- **Certifications** — AZ-900 (fundamentals), AZ-104 (admin), AZ-305 (architect), SC-100 (security)
- **Hands-on labs** — sandbox environments for practical experience
- **Partner training** — accelerated enablement through Microsoft partners

### Change Management

Cloud adoption is an organizational change, not just a technology project:

- **Executive sponsorship** — visible, active support from senior leadership
- **Communication plan** — regular updates on progress, wins, and lessons learned
- **Quick wins** — demonstrate value early to build momentum and trust
- **Feedback loops** — channels for teams to raise concerns and suggest improvements
- **Celebrate success** — recognize teams and individuals driving adoption
- **Address resistance** — understand root causes (fear, skill gaps, loss of control)
