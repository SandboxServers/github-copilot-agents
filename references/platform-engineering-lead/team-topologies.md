## Team Topologies for Platform Engineering

### Platform Team as Enabling Team
The platform team's purpose is to enable stream-aligned teams to deliver faster.
The platform team is not a gatekeeper, approval authority, or bottleneck.
It is a product team whose customers are other engineering teams.

Core responsibilities:
- Build and maintain the internal developer platform
- Provide golden paths for common development scenarios
- Operate shared infrastructure and platform services
- Reduce cognitive load on application teams
- Encode governance into automation rather than manual processes

### Stream-Aligned Teams as Customers
Stream-aligned teams are the platform's primary customers. They:
- Build and operate applications that deliver business value
- Consume platform services through self-service interfaces
- Provide feedback that drives platform improvement
- Operate autonomously within the guardrails the platform provides
- Own the end-to-end lifecycle of their applications

The platform succeeds when stream-aligned teams can deliver faster,
with fewer dependencies, and without sacrificing quality or compliance.

### Complicated-Subsystem Teams
Some domains require deep specialized expertise that doesn't belong in the platform
team or stream-aligned teams:
- Database performance optimization
- Machine learning infrastructure
- Security operations and incident response
- Networking architecture for complex topologies

These teams provide expertise as a service, reducing the knowledge burden
on other teams while maintaining deep capability in critical areas.

### Thin Platform Principle
Build the thinnest viable platform — the minimum set of capabilities that
reduces the most cognitive load across the most teams.

Anti-patterns of a thick platform:
- Monolithic platform that tries to do everything
- Long feature backlogs with no adoption validation
- Platform team becomes larger than the teams it serves
- Custom solutions for problems that commercial products solve better

Healthy platform indicators:
- Small, focused team delivering high-impact capabilities
- High adoption rates with minimal support burden
- Clear boundaries between platform and application responsibilities
- Regular deprecation of features that aren't used

### Cognitive Load as Primary Metric
The platform's primary value is reducing cognitive load on application teams.
Measure this by tracking:
- How many distinct tools and systems must a developer understand?
- How many manual steps are required for common workflows?
- How much documentation must a developer read to be productive?
- How often do developers need to ask for help from the platform team?

If the platform adds complexity rather than removing it, something is wrong.

### APIs Between Teams, Not Tickets
Team interactions should be through well-defined interfaces:

| Instead of... | Use... |
|---|---|
| Filing a ticket for infrastructure | Self-service provisioning API |
| Emailing the platform team for access | Automated access request workflow |
| Scheduling a meeting to discuss requirements | Service catalog with clear documentation |
| Waiting for manual approval | Policy-as-code with automated evaluation |
| Ad-hoc Slack questions | Searchable documentation and FAQs |

The goal is to minimize synchronous dependencies between teams.
Every ticket represents a missing platform capability or a gap in self-service.
