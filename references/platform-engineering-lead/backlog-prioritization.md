## Platform Backlog Prioritization

### Managing the Platform Backlog
The platform backlog is not a wish list — it's a prioritized queue of investments
that deliver measurable value to development teams. Every item should be traceable
to a real problem experienced by real teams.

### Prioritization Criteria

| Criterion | Question to Ask | Weight |
|---|---|---|
| **Adoption Impact** | How many teams will use this? | High |
| **Toil Reduction** | How many manual hours does this eliminate? | High |
| **Risk Reduction** | Does this prevent security, compliance, or availability issues? | High |
| **Developer Satisfaction** | Does this address a top friction point? | Medium |
| **Strategic Alignment** | Does this advance platform maturity goals? | Medium |
| **Maintenance Cost** | What's the ongoing cost to support this? | Medium |

### Opportunity Cost Thinking
Every feature you build has an opportunity cost — the value of the features you
didn't build instead. Ask:
- If we build this, what gets delayed?
- Is this the highest-impact use of the team's time?
- Could we achieve 80% of the value with 20% of the effort?

### Saying No to Bespoke Requests
Platform teams face constant pressure for one-off customizations.

Reasons to say no:
- Serves only one team while the backlog has items serving many
- Creates maintenance burden disproportionate to value
- Undermines the golden path by creating parallel workflows
- Can be solved by the requesting team within existing guardrails

How to say no constructively:
- Acknowledge the need and explain the trade-off
- Offer an alternative using existing capabilities
- Add the request to the backlog for future evaluation if demand grows
- Help the team self-serve a solution within platform boundaries

### Decision Template

For each backlog item, document:

| Field | Description |
|---|---|
| **Problem Statement** | What pain are developers experiencing? |
| **Affected Teams** | How many teams and how frequently? |
| **Proposed Solution** | What will the platform provide? |
| **Impact** | Quantify time saved, risk reduced, or adoption enabled |
| **Effort** | T-shirt size estimate (S/M/L/XL) |
| **Decision** | Prioritize / Defer / Decline |
| **Rationale** | Why this decision? |

### Review Cadence
- **Weekly** — review new requests and triage
- **Bi-weekly** — reprioritize backlog based on new data
- **Monthly** — review adoption metrics for recently shipped features
- **Quarterly** — assess backlog against platform maturity roadmap
