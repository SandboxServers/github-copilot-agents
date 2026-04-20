## Build vs Buy vs Adopt

### Decision Framework
Every platform capability requires a build, buy, or adopt decision.
Getting this wrong wastes time and money. Getting it right accelerates delivery.

### When to Build
Build custom solutions when:
- The capability provides **unique competitive advantage**
- Deep customization is required that no product supports
- No existing product fits the specific workflow
- The team has capacity to build AND maintain long-term
- Integration requirements are too complex for off-the-shelf

Risks of building:
- Ongoing maintenance burden falls entirely on the platform team
- Key-person dependency — if the builder leaves, who maintains it?
- Opportunity cost — time spent building is time not spent on higher-value work

### When to Buy
Buy commercial products when:
- The capability is a **commodity** (CI/CD, monitoring, secret management)
- Faster time to value is critical
- The vendor maintains updates, patches, and compliance
- The total cost of ownership is lower than building
- The product integrates well with existing platform components

Risks of buying:
- Vendor lock-in and dependency on vendor roadmap
- License costs scale with usage
- Customization may be limited or expensive

### When to Adopt (Open Source)
Adopt open-source solutions when:
- A mature, community-maintained project exists
- Avoiding vendor lock-in is a priority
- The team can customize incrementally as needs evolve
- The community is active with regular releases and security patches
- The organization can contribute back to sustain the project

Risks of adopting:
- Abandonment risk if the community loses momentum
- Security patches depend on community responsiveness
- Integration and support costs are borne internally

### Common Platform Components

| Component | Typical Decision | Reasoning |
|---|---|---|
| CI/CD pipeline | Buy (GitHub Actions, ADO Pipelines) | Commodity capability, well-supported |
| Infrastructure as Code | Adopt (Terraform) or Buy (Bicep) | Mature tooling, strong community/vendor |
| Monitoring/Observability | Buy (Azure Monitor, Datadog) | Deep integration, vendor-maintained |
| Secret Management | Buy (Azure Key Vault) | Security-critical, vendor-maintained |
| Service Mesh | Adopt (Istio) or Buy (vendor mesh) | Depends on team expertise |
| Internal Portal | Build or Adopt (Backstage) | Unique to org, customization needed |
| Policy Engine | Buy (Azure Policy) + Adopt (OPA) | Combination often necessary |
| Template Library | Build | Specific to organization's standards |

### Decision Template
For each capability decision, document:
1. **Problem** — what need does this address?
2. **Options evaluated** — what did you consider?
3. **Decision** — build, buy, or adopt?
4. **Reasoning** — why this option over others?
5. **Trade-offs accepted** — what are we giving up?
6. **Review date** — when will we reassess this decision?
