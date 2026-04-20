## Anti-Patterns in Azure Specialty Coordination

These anti-patterns emerge when the Specialty Lead fails to coordinate effectively — or when specialists operate without coordination at all.

### Local Optimization at Global Cost

Specialists naturally optimize for their domain. Without coordination, this produces systems that are locally optimal and globally broken.

- **Database specialist picks Cosmos DB multi-region write** for maximum availability — but the network team hasn't designed for the cross-region traffic, the cost analyst hasn't budgeted for multi-region RU consumption, and the application team doesn't know how to handle conflict resolution.
- **Network specialist segments everything into micro-subnets** for security isolation — then compute can't deploy because there aren't enough IPs, and integration patterns break because services can't reach each other.
- **Storage specialist enables immutable storage** for compliance — then the application team discovers they need to update records and there's no path forward.

**Fix:** Every specialist design must include a "blast radius" section: what other domains does this decision constrain or enable?

### No Cross-Domain Dependency Mapping

Specialists design in isolation. Nobody maps how Domain A's decisions affect Domain B. You discover dependencies in production when things break.

- Network topology changes that silently break database connectivity.
- Identity model changes that revoke access patterns other domains depend on.
- Monitoring agent deployments that consume compute resources nobody budgeted for.

**Fix:** Maintain a dependency matrix. Before any design is approved, trace its impact across all connected domains.

### Sequential Engagement When Parallel Works

Waiting for the network design to be "complete" before engaging compute, then waiting for compute before engaging database. This waterfall approach doubles timelines and produces designs where later specialists are constrained by earlier decisions they had no input on.

**Fix:** Engage all relevant specialists at the design phase. Use parallel workstreams with defined interface contracts. The network team doesn't need to finish — they need to commit to an interface.

### Over-Specialization

Every question gets routed to a specialist, even when the answer is straightforward. "Should I use a Standard_D4s_v5 or Standard_D8s_v5?" doesn't need the compute engineer for a non-critical workload. Over-routing creates bottlenecks and trains teams to be helpless.

**Fix:** Define a complexity threshold. Simple, well-documented decisions stay with the requesting team. Specialists engage for novel problems, cross-domain concerns, or production-critical workloads.

### No Shared Architectural Principles

Each specialist follows their own domain best practices but there are no cross-domain principles. The network team values isolation, the compute team values simplicity, the database team values performance. These principles conflict and nobody has reconciled them.

**Fix:** Establish shared principles that all specialists agree to. WAF provides the framework. "We optimize for operational excellence first, then reliability, then cost" — a shared priority order resolves conflicts before they start.

### Ignoring Cost When Designing for Reliability or Performance

The reliability specialist adds multi-region failover. The performance specialist adds premium tiers everywhere. The security specialist adds dedicated HSMs. Each addition is justified in isolation. Together, they triple the monthly spend.

**Fix:** Cost review is mandatory on every architecture change. The Cost Optimization Specialist has standing invitation to every design review. No design is approved without a cost estimate.

### Security as End-Gate Instead of Continuous Input

The architecture is designed, the specialists have agreed on all the components, and then security reviews it. Security finds problems that require fundamental redesign. Weeks of work are invalidated.

**Fix:** Security specialist provides input from day one. Security requirements are constraints that shape the design, not a checklist applied after the design is complete.

### Not Validating Architecture with Implementers

The Specialty Lead designs a beautiful multi-domain architecture on the whiteboard. The specialists who will actually implement it weren't in the room. They discover practical problems during implementation that the design didn't account for.

**Fix:** Every architecture must be reviewed by the specialists who will implement it before it's approved. "Can you actually build this?" is a required question.

### Treating Monitoring as Afterthought

The architecture is designed, built, and deployed. Then someone asks "how do we monitor this?" Monitoring is bolted on instead of built in. Observability gaps appear in production.

**Fix:** The Monitoring Engineer is engaged at design time. Every component design includes: what metrics matter, what alerts fire, what dashboards show, what logs are needed. Monitoring is architecture, not decoration.

### No WAF Alignment

Designing without considering all five Well-Architected Framework pillars. A solution optimized for reliability that ignores operational excellence. A cost-optimized design that ignores security. Pillar-blind design creates technical debt.

**Fix:** Every architecture review scores against all five WAF pillars. Trade-offs between pillars are explicit and documented, not accidental.

### Handoff Failures

The network team designs a topology and hands it to the compute team. The compute team can't implement it because the design assumed capabilities that don't exist, used terminology differently, or didn't account for compute-side constraints.

**Fix:** Handoffs use interface contracts with explicit assumptions, constraints, and acceptance criteria. Both sides sign off. "I designed it" and "I can implement it" must both be true.

### No Shared Vocabulary

The network team says "endpoint," the compute team says "endpoint," the integration team says "endpoint." They all mean different things. Miscommunication creates design errors that surface in production.

**Fix:** Maintain a shared glossary. When terms are ambiguous, qualify them: "private endpoint (networking)," "service endpoint (integration)," "compute endpoint (API)." Precision in language prevents precision problems in architecture.
