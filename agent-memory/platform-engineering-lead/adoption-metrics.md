## Platform Adoption Metrics

### Why Measure Adoption
Features don't matter if nobody uses them. The platform's value is realized only
through adoption. Measuring adoption reveals whether the platform is solving real
problems and whether investment is being directed to the right areas.

### Core Metrics

#### Adoption Rate
Percentage of teams actively using golden paths and platform services.

| Measurement | Target |
|---|---|
| Teams on golden path for new services | >80% |
| Teams using self-service provisioning | >70% |
| Teams on standard CI/CD pipeline | >90% |
| Teams using platform observability stack | >75% |

Track adoption rate over time. Stagnation or decline signals a problem
with the platform, not with the teams.

#### Time to First Deployment
How long it takes a new service to reach production from project kickoff.

| Maturity | Typical Time |
|---|---|
| No platform | 4-8 weeks |
| Basic platform | 1-2 weeks |
| Mature platform | 1-3 days |
| Optimized platform | Hours |

This metric captures the end-to-end value of the platform — infrastructure,
CI/CD, security, and operational readiness combined.

#### Developer Satisfaction
Regular surveys measuring developer experience with the platform.

Methods:
- **NPS (Net Promoter Score)** — "How likely are you to recommend this platform to a colleague?"
- **Satisfaction survey** — quarterly, covering specific capabilities
- **Friction log** — developers document pain points during real workflows
- **Office hours feedback** — qualitative insights from support interactions

#### Toil Reduction
Hours saved through platform automation compared to manual processes.

| Process | Manual Time | Automated Time | Savings per Instance |
|---|---|---|---|
| New environment provisioning | 2-5 days | 15 minutes | ~20-40 hours |
| Access request fulfillment | 1-3 days | 5 minutes | ~8-24 hours |
| Security compliance check | 1 week | Continuous | ~40 hours |
| New service deployment setup | 2-3 days | 30 minutes | ~16-24 hours |

Multiply savings per instance by frequency to calculate total toil reduction.

#### Self-Service Ratio
Percentage of developer requests fulfilled without human intervention.

| Maturity | Self-Service Ratio |
|---|---|
| Ticket-based | <10% |
| Partial automation | 30-50% |
| Platform-driven | 70-85% |
| Fully self-service | >90% |

Every request that requires a human is a candidate for automation.

#### Support Ticket Volume
Track platform-related support tickets over time.

Healthy trend: volume decreases as the platform matures.
Unhealthy trend: volume increases despite new features (signals usability problems).

Categorize tickets to identify:
- Missing self-service capabilities
- Documentation gaps
- Usability issues with existing features
- Edge cases the golden paths don't cover

### Reporting and Action
- **Dashboard** — real-time metrics visible to the platform team and stakeholders
- **Monthly review** — discuss trends, identify areas needing attention
- **Quarterly report** — executive summary linking metrics to business outcomes
- **Action items** — every metric that trends poorly should generate a backlog item
