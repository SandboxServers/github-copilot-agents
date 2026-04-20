## Platform Engineering Best Practices

### Overview
These best practices draw from Microsoft's platform engineering guidance, the Platform
Engineering Capability Model, and patterns observed across successful internal developer
platforms. They focus on what works in practice, not just in theory.

> **Source:** Microsoft Platform Engineering
> ([principles](https://learn.microsoft.com/platform-engineering/about/principles),
> [product mindset](https://learn.microsoft.com/platform-engineering/about/product-mindset),
> [self-service](https://learn.microsoft.com/platform-engineering/developer-self-service)).

### Treat the Platform as a Product with Internal Customers
- Developers are customers, not users — understand their needs through research
- Assign a product manager (or product-minded engineer) to the platform team
- Maintain a public roadmap and accept feature requests through a backlog
- Prioritize based on developer impact, not technical elegance
- Measure success by adoption and satisfaction, not features shipped

### Start with Golden Paths for the Most Common Use Cases
- Identify the top 3 developer journeys (new service, new environment, new pipeline)
- Build opinionated, end-to-end golden paths for each
- Make the right thing the easy thing — defaults should encode best practices
- Provide documented escape hatches for teams with legitimate edge cases
- Iterate on paths based on deviation rates and developer feedback

### Measure What Matters
- **DORA metrics:** Deployment frequency, lead time, change failure rate, MTTR
- **Developer satisfaction (DevEx):** Survey-based + behavioral telemetry
- **Time-to-first-deployment:** How fast can a new team ship to production?
- **Adoption rate:** Percentage of teams using golden paths voluntarily
- **Toil reduction:** Hours saved per developer per week on platform-automated tasks
- Report outcomes quarterly to leadership to justify continued investment

### Self-Service: Developers Should Never Wait for the Platform Team
- Every recurring ticket is a candidate for self-service automation
- Environment provisioning, secret management, and DNS should be instant
- Use guardrails (Azure Policy, OPA) instead of approval workflows
- Audit after the fact rather than gate before the fact
- Target: developers can go from idea to running code without filing a ticket

### Documentation: Every Golden Path Has a Quick-Start Guide
- Treat documentation as part of the feature — incomplete docs means the feature isn't done
- Quick-start guides: get developers to "hello world" in under 15 minutes
- Reference docs: cover every configuration option and escape hatch
- Runbooks: what to do when things go wrong
- Keep docs next to code — versioned, reviewed, and tested

### Feedback Loops: Regular Developer Interviews and Surveys
- Quarterly developer satisfaction surveys (keep them short — 5-7 questions)
- Monthly office hours where developers can ask questions and share pain
- Embed platform liaisons in stream-aligned teams for continuous feedback
- Act on feedback visibly — publish what changed because of developer input
- Combine qualitative feedback with quantitative telemetry for full picture

### Iterate: Start Simple, Add Capabilities Based on Demand
- Ship the thinnest viable platform (TVP) first — not a grand architecture
- Add capabilities only when multiple teams request them
- Deprecate features that aren't adopted — reduce surface area
- Run build-measure-learn cycles on a 2-4 week cadence
- Resist the urge to build for imagined future scale

### Developer Portal for Discoverability
- Use Backstage, Port, or a similar developer portal for a unified interface
- Catalog all services, APIs, golden paths, and documentation in one place
- Enable search and discovery — developers shouldn't need to ask where things are
- Integrate with existing tools (GitHub, Azure DevOps, Jira) rather than replacing them
- Keep the portal fresh — stale data erodes trust faster than no portal

### Inner Source: Enable Contributions from Outside the Platform Team
- Use a provider/plugin model so other teams can extend the platform
- Define clear contribution guidelines and review processes
- Celebrate contributions publicly to encourage participation
- Maintain ownership boundaries — contributors build, platform team reviews
- Don't require contributors to maintain their contributions indefinitely

### Platform Reliability: Treat It Like Production
- Define SLOs for every platform service (portal uptime, provisioning latency)
- Monitor platform services with the same rigor as production workloads
- Run incident reviews when platform outages impact developer productivity
- Budget for on-call and maintenance, not just feature development
- Communicate planned maintenance windows to all platform consumers

### Governance Built In, Not Bolted On
- Embed policy-as-code into golden paths from day one
- Use guardrails that guide developers toward compliance automatically
- Make the compliant path the easiest path — not a separate approval workflow
- Integrate security scanning, cost controls, and tagging into provisioning
- Grandfather existing deployments with clear migration timelines

### Summary
The best platforms share a common thread: they treat developer experience as a
first-class product concern, measure outcomes relentlessly, and iterate based on
what developers actually need — not what the platform team assumes they need.
