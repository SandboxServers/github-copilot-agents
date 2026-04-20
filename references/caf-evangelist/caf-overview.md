## Cloud Adoption Framework — Overview

### What Is CAF

The Microsoft Cloud Adoption Framework (CAF) is a collection of documentation, guidance,
best practices, and tools that provides a proven approach for cloud adoption. It covers
the full lifecycle from initial strategy through ongoing operations.

### The 7 Phases

CAF is organized into seven phases. The first four are roughly sequential; the last three
operate in parallel and continuously alongside adoption.

```
Sequential Foundation              Continuous Operations
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   ┌──────────┐ ┌──────────┐ ┌──────────┐
│ Strategy  │→│   Plan   │→│  Ready   │→│  Adopt   │   │  Govern  │ │  Secure  │ │  Manage  │
└──────────┘ └──────────┘ └──────────┘ └──────────┘   └──────────┘ └──────────┘ └──────────┘
```

### Phase Summaries

**Strategy** — Define motivations for cloud adoption (cost savings, agility, scale,
innovation). Identify measurable business outcomes. Build a financial model with TCO and
ROI analysis. Select the first adoption project.

**Plan** — Rationalize the digital estate using the 5 Rs (Rehost, Refactor, Rearchitect,
Rebuild, Replace). Build the cloud adoption plan with timelines and milestones. Assess
skills readiness across the organization. Align teams to the operating model.

**Ready** — Prepare the Azure environment through landing zone deployment. Ensure the
foundation is in place before workloads arrive: networking, identity, governance policies,
monitoring, and security controls. Address skills readiness gaps.

**Adopt** — Execute cloud migration and innovation projects. Migration moves existing
workloads using the 8 Rs. Innovation builds new cloud-native capabilities. Both tracks
can run in parallel across different workload waves.

**Govern** — Establish and enforce cloud governance across five disciplines:
- Cost Management — budgets, alerts, tagging, chargeback
- Security Baseline — Microsoft Cloud Security Benchmark, Defender for Cloud
- Identity Baseline — Entra ID, conditional access, PIM
- Resource Consistency — naming conventions, tagging, allowed regions, approved SKUs
- Deployment Acceleration — IaC standards, CI/CD pipelines, policy-as-code

**Secure** — Implement security across three pillars:
- Risk insights — threat modeling, security posture assessment
- Business resilience — disaster recovery, backup, incident response
- Asset protection — network security, data protection, endpoint security

**Manage** — Deliver operational excellence through:
- Business commitments — SLAs, SLOs, operational agreements
- Operational compliance — patching, configuration management, change control
- Workload operations — monitoring, alerting, performance optimization

### Phases Are Not Linear

CAF is iterative by design. Organizations revisit earlier phases as they learn:
- Governance matures as new workloads are adopted
- Strategy evolves as business conditions change
- Landing zones expand as new requirements emerge
- Security posture strengthens continuously

The key insight: start where you are, use what applies, iterate frequently. CAF is a
framework to draw from, not a checklist to complete sequentially.

### When to Reference Each Phase

| Situation | Start Here |
|-----------|-----------|
| No cloud strategy exists | Strategy |
| Strategy defined but no plan | Plan |
| Plan exists but no Azure environment | Ready |
| Landing zone deployed, moving workloads | Adopt |
| Workloads in cloud but uncontrolled | Govern |
| Security gaps or compliance requirements | Secure |
| Operational issues, no monitoring | Manage |
