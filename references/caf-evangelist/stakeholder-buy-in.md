## Stakeholder Buy-In Strategies

### Translating for Different Audiences

| Audience | They Care About | CAF Speaks To Them Via |
|----------|----------------|----------------------|
| **Executives / CFO** | Cost, risk, time-to-value | Strategy phase: business case, ROI, cost optimization |
| **Security / CISO** | Compliance, threat surface, access control | Secure methodology + Governance disciplines |
| **Finance / FinOps** | Budget predictability, chargeback, waste reduction | Cost Management discipline, Azure Cost Management |
| **Operations / SRE** | Uptime, monitoring, incident response | Manage methodology (RAMP), monitoring design area |
| **Developers / DevOps** | Speed, autonomy, tooling, CI/CD | Platform automation design area, subscription vending |
| **Compliance / Legal** | Regulatory adherence, audit readiness | Governance methodology, Azure Policy compliance |

### Common Resistance Patterns and Responses

| Resistance | Root Cause | Response |
|-----------|-----------|----------|
| "This is too much process" | Fear of slowdown | Show how guardrails enable speed (self-service subscription vending vs. ticket-based provisioning) |
| "We already have our own way" | Silo / fiefdom | Acknowledge what works, show where gaps exist, propose incremental adoption |
| "We don't need a landing zone" | Urgency / impatience | Show the cost of remediation when governance is bolted on later (real examples) |
| "Cloud is just someone else's computer" | Fear of change | Start with hybrid scenarios (Arc, AVS) that extend existing investments |
| "We can't afford to stop and plan" | Ongoing project pressure | CAF doesn't require stopping — it's iterative. Start with the next workload. |

## Assessment Questions This Evangelist Always Asks

### Cloud Maturity
- Where are you today? (No cloud, experimenting, some workloads, majority in cloud)
- What's the business driver? (Datacenter exit, cost reduction, innovation, compliance, M&A)
- Who owns cloud today? (One team? Nobody? Everyone?)
- What's your governance posture? (Policies in place? Audit only? Nothing?)

### Organizational Readiness
- Do you have a cloud strategy team with executive sponsorship?
- Is there a dedicated cloud platform team or is it "whoever has time"?
- What's the relationship between central IT and application teams? (Collaborative? Adversarial? Non-existent?)
- Are developers deploying directly to Azure or going through a ticket system?

### Technical Foundation
- Do you have an Azure landing zone? (If yes, what implementation? If no, how are subscriptions organized?)
- How many Azure subscriptions do you have? How are they organized?
- What's your network topology? (Hub-spoke, VWAN, flat, none?)
- How do you enforce standards? (Azure Policy, manual review, nothing?)
- What IaC tooling do you use? (Terraform, Bicep, ARM templates, ClickOps?)

### Probing for Gaps
- What happens when a team needs a new Azure subscription? (If > 2 weeks → subscription vending problem)
- How do you know if someone deploys a non-compliant resource? (If "we don't" → governance gap)
- What's your break-glass procedure for emergency access? (If blank stare → identity gap)
- How do you track cloud costs by team/project? (If "we don't" → cost management gap)
- When was the last time you reviewed who has Owner/Contributor on production subscriptions? (If "never" → security gap)
