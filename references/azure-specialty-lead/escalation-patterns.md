## Escalation Patterns

Not every disagreement or blocker needs the Specialty Lead. But some do. These patterns define when and how to escalate.

### Specialist Disagrees with Another Specialist

**Situation:** Integration architect wants synchronous API calls between services. Compute engineer says the latency budget doesn't support synchronous hops across three services.

**Escalation:** Azure Specialty Lead arbitrates. Both specialists present their constraints. The Specialty Lead evaluates system-level impact and makes the call — or proposes a hybrid (synchronous for the critical path, asynchronous for the rest).

**Principle:** The specialist with the broader system constraint usually wins. Latency budgets are harder to fix than integration patterns.

### Cross-Cutting Concern Creates Conflict

**Situation:** Network engineer requires all traffic through Azure Firewall. AI specialist says the GPU inference endpoint needs direct internet egress for model updates with latency that firewall inspection breaks.

**Escalation:** Design session with network engineer, AI specialist, and Specialty Lead. Explore options: firewall rule exception for specific endpoints, asymmetric routing, scheduled model updates during maintenance windows.

**Principle:** Cross-cutting concerns (networking, identity, security) are constraints, not suggestions. Exceptions require justification and compensating controls.

### Performance vs Cost Trade-Off

**Situation:** Database specialist recommends Cosmos DB with multi-region writes for sub-10ms latency. Cost analysis shows this is 4x the budget for the data tier.

**Escalation:** Include cost optimization specialist, database specialist, and compute engineer. Explore: Can application-level caching reduce database pressure? Is multi-region write necessary or would single-write with read replicas suffice? Can the latency target be relaxed?

**Principle:** Cost is a constraint like any other. The cheapest solution that meets requirements wins — not the most technically elegant one.

### Security Finding Blocks Deployment

**Situation:** Security analyst finds that the proposed architecture uses public endpoints for storage accounts. The deployment timeline is tight.

**Escalation:** Security analyst and affected specialist (storage engineer, plus compute if VNet integration changes are needed). Evaluate: Can private endpoints be added without redesign? What's the actual risk of a short-term exception with compensating controls (IP restrictions, SAS tokens with short expiry)?

**Principle:** Security findings with Critical or High severity block deployment. No exceptions without documented risk acceptance from the workload owner and security team.

### Incident Reveals Architectural Gap

**Situation:** Production incident caused by DNS resolution failure — private endpoint DNS zone wasn't linked to the spoke VNet where a new compute resource was deployed.

**Escalation:** Post-incident review with network engineer, compute engineer, and Specialty Lead. Root cause: handoff gap — network provided DNS zones but didn't validate new spoke VNet linkage. Fix: Add DNS zone linkage to deployment checklist and post-deployment validation.

**Principle:** Incidents that cross specialty boundaries indicate a coordination gap, not a specialist failure. Fix the process, not the person.

### When NOT to Escalate

- Single-domain technical question → Specialist handles it
- Well-understood trade-off with clear precedent → Specialist decides using ADR
- Routine configuration or sizing → Specialist's expertise
- The decision is easily reversible → Just decide and observe

Escalate when the decision is irreversible, crosses domains, or when specialists have conflicting constraints with no obvious resolution.
