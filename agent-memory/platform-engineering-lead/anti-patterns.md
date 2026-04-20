## Anti-Patterns in Platform Engineering

### Overview
Platform engineering initiatives fail in predictable ways. These broader anti-patterns
capture organizational, strategic, and execution failures that undermine internal
developer platforms. Recognizing them early saves months of wasted investment
and prevents erosion of developer trust.

> **Source:** Patterns informed by Microsoft Platform Engineering guidance
> ([principles](https://learn.microsoft.com/platform-engineering/about/principles),
> [product mindset](https://learn.microsoft.com/platform-engineering/about/product-mindset),
> [capability model](https://learn.microsoft.com/platform-engineering/platform-engineering-capability-model)).

### Building Without Talking to Developers
- Platform roadmap driven by assumptions, not developer research
- No customer interviews, no shadowing, no observation of real workflows
- Features designed for how leadership imagines developers work
- **Fix:** Adopt customer development practices. Interview at least 5 teams before each quarter's planning.
  Observe developers using the platform in real time.

### Mandating Platform Adoption Instead of Making It Attractive
- Leadership decrees that all teams must use the platform
- Compliance is superficial — teams use it minimally and resent it
- Adoption numbers look good on dashboards but satisfaction is low
- **Fix:** Make the platform so good teams choose it voluntarily. Mandate only
  baseline security and compliance — never tooling preferences.

### Over-Engineering Before Validating with Real Users
- Months of architecture before any team touches the platform
- Complex abstractions serve imagined future scenarios, not current pain
- "We'll need this at scale" justifies features no one uses today
- **Fix:** Ship a thinnest viable platform (TVP) early. Get real users.
  Build for today's needs with extension points for tomorrow.

### No Golden Paths — Platform Without Opinionated Defaults
- Platform provides primitives but no recommended paths
- Every team must assemble their own workflow from building blocks
- Cognitive load remains high despite the platform's existence
- **Fix:** Create opinionated, end-to-end golden paths for the top 3 use cases.
  Make the right thing the easy thing.

### Treating Platform as a Project, Not a Product
- Platform has a launch date and a "done" milestone
- After launch, the team moves on or is disbanded
- No roadmap, no iteration, no ongoing investment
- **Fix:** Fund the platform as a product with a permanent team, a backlog,
  and quarterly planning tied to developer outcomes.

### No Self-Service — Developers Still File Tickets
- Common operations require tickets to the platform team
- Provisioning an environment takes days instead of minutes
- The platform adds process rather than removing it
- **Fix:** Every recurring request is a candidate for self-service automation.
  If developers wait on you, the platform is failing.

### Platform Team as Bottleneck Instead of Enabler
- Platform team controls access, approvals, and configuration changes
- Teams must justify needs to the platform team before proceeding
- Innovation slowed by permission-seeking and queue times
- **Fix:** Provide guardrails for safe self-service. Audit after the fact
  rather than approve before. Trust teams to operate within boundaries.

### Not Measuring Platform Adoption or Developer Satisfaction
- No metrics on adoption rate, developer satisfaction, or toil reduction
- Success defined by features shipped rather than outcomes delivered
- No data to justify continued investment to leadership
- **Fix:** Define success metrics before building. Track DORA metrics,
  developer satisfaction (DevEx), and time-to-first-deployment from day one.

### Ignoring Existing Tools That Work
- Replacing tools developers already use and like
- Migrations justified by consistency rather than improvement
- Teams lose productivity during transitions with no net gain
- **Fix:** Integrate with existing tools where possible. Only replace when
  the new option is measurably better for the developer experience.

### No Documentation or Onboarding for Platform Features
- Features exist but developers don't know about them
- Onboarding requires tribal knowledge or Slack messages
- Error messages are cryptic and unhelpful
- **Fix:** Every golden path has a quick-start guide. Treat docs as part of
  the feature — incomplete docs means the feature isn't done.

### Platform Sprawl — Too Many Features, None Fully Baked
- Platform tries to solve everything at once
- Half-finished capabilities create more confusion than value
- Teams don't trust any single feature because none work reliably
- **Fix:** Finish fewer things well. A polished golden path for one use case
  beats five incomplete paths that need workarounds.

### No SLOs for Platform Services
- Platform services have no reliability commitments
- Developers don't know what uptime or response time to expect
- Platform outages are treated with less urgency than production
- **Fix:** Define SLOs for every platform service. Treat platform reliability
  as seriously as production reliability — developers depend on it.

### Summary
Most anti-patterns share a root cause: treating the platform as a technical
project rather than a product serving developer customers. Apply product thinking —
understand your users, measure outcomes, iterate based on feedback — and most
anti-patterns resolve themselves.
