## Platform Anti-Patterns

### Recognizing What Goes Wrong
Platform engineering initiatives fail when teams fall into predictable traps.
Recognizing these anti-patterns early prevents wasted investment and
loss of developer trust.

### Anti-Pattern: Building Features Nobody Asked For
- Platform team builds based on assumptions, not research
- Features ship without validating demand
- Roadmap driven by technology excitement rather than developer pain
- **Fix:** Talk to developers. Observe their workflows. Prioritize based on data.

### Anti-Pattern: Mandating Adoption
- Leadership mandates that all teams must use the platform
- Teams comply minimally and resent the platform
- Adoption numbers look good but satisfaction is low
- **Fix:** Make the platform so good that teams choose it voluntarily.
  Mandate only baseline security and compliance — never tooling preferences.

### Anti-Pattern: Becoming a Bottleneck
- Platform team becomes a dependency for common operations
- Developers wait for the platform team to fulfill requests
- The platform creates more process than it eliminates
- **Fix:** Every recurring request is a candidate for self-service automation.
  If teams are waiting on you, the platform isn't working.

### Anti-Pattern: Over-Engineering
- Platform designed for scale it doesn't need yet
- Complex abstractions that serve future scenarios, not current ones
- Months of development before any team can use anything
- **Fix:** Start with the thinnest viable platform. Ship early. Iterate based on feedback.
  Build for today's needs with extension points for tomorrow.

### Anti-Pattern: Ignoring Developer Experience
- Platform works technically but is painful to use
- Documentation is incomplete, outdated, or absent
- Error messages are cryptic and unhelpful
- Onboarding requires tribal knowledge
- **Fix:** Treat developer experience as a first-class product concern.
  Invest in UX, documentation, error messages, and onboarding flows.

### Anti-Pattern: Platform Team as Gatekeepers
- Platform team controls access and approvals
- Teams must justify their needs to the platform team
- Innovation is slowed by permission-seeking
- **Fix:** Provide guardrails that allow safe self-service. Trust teams to operate
  within boundaries. Audit after the fact rather than approve before.

### Anti-Pattern: No Feedback Loop
- Platform team ships features and moves on
- No mechanism to learn whether features are used or valued
- Developer complaints are dismissed as resistance to change
- **Fix:** Establish regular feedback channels — surveys, office hours, embedded liaisons.
  Act on feedback visibly to build trust.

### Anti-Pattern: Building Without Measuring
- No metrics on adoption, satisfaction, or toil reduction
- Success defined by features shipped rather than outcomes delivered
- No way to justify continued investment to leadership
- **Fix:** Define success metrics before building. Track adoption from day one.
  Report outcomes, not outputs.

### Summary
Every anti-pattern has a common root cause: treating the platform as a technical
project rather than a product serving developer customers. Apply product thinking —
understand your users, measure outcomes, iterate based on feedback — and most
anti-patterns resolve themselves.
