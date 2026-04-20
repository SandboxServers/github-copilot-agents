## Best Practices for Azure Specialty Coordination

These practices produce architectures where the whole is greater than the sum of the specialists' parts.

### Architecture Reviews with All Relevant Specialists

Never design in isolation. When a solution touches multiple domains, every affected specialist must be in the review.

- **Before the review:** share the proposed architecture async so specialists can prepare questions about their domain's interface points.
- **During the review:** walk through the solution domain by domain. At each boundary, both sides confirm: "this is what I'm providing" and "this is what I'm consuming."
- **After the review:** publish the agreed interfaces and constraints. Any change to an interface triggers re-review with affected specialists.

The Specialty Lead facilitates but does not dominate. The goal is cross-domain agreement, not top-down dictation.

### WAF as Shared Framework

The Well-Architected Framework provides a common language and common priorities across all specialties. Use it as the coordination framework, not just an assessment tool.

- **Agree on pillar priority order** for each workload. A financial trading system prioritizes reliability over cost. An internal dev/test environment prioritizes cost over performance. Make this explicit.
- **Score designs against all five pillars.** Every specialist reviews how their domain contributes to (or detracts from) each pillar.
- **Document trade-offs.** When optimizing for one pillar hurts another, record why the trade-off was made and what the accepted risk is.

### Dependency Mapping

Understand how domains connect before designing within them.

- **Maintain a service dependency matrix.** Rows are services, columns are the domains they depend on. Update it with every architecture change.
- **Identify critical paths.** Which dependency chains, if broken, bring down the entire system? Those chains get extra validation and monitoring.
- **Map deployment order.** Some services must exist before others can be configured. Identity before RBAC. Networking before compute. DNS before private endpoints. Get the order wrong and deployments fail.

### Cost Review on Every Architecture Change

Cost is not a phase — it's a continuous constraint.

- **Every design change includes a cost delta.** "Adding read replicas costs ~$X/month." No design is approved without a cost estimate.
- **The Cost Optimization Specialist reviews monthly spend** against the architecture budget. Drift is flagged immediately, not at quarterly review.
- **Reserved capacity and savings plans** are coordinated across domains. The compute team's reservations and the database team's reserved capacity should align to the same commitment horizon.

### Security by Design

Security is a design constraint, not a review gate.

- **Security specialist provides requirements at project kickoff.** Encryption standards, network isolation requirements, identity model, compliance constraints — these shape the design from day one.
- **Every specialist designs for least privilege.** Not just in their domain — across domain boundaries. The compute workload's managed identity gets exactly the database permissions it needs, no more.
- **Threat modeling spans domains.** An attack path rarely stays in one domain. "Compromised compute identity → database access → data exfiltration" is a cross-domain threat that no single specialist would catch alone.

### Monitoring Integrated into Architecture

Monitoring is not bolted on after deployment. It is part of the architecture.

- **Every component design includes observability requirements.** What metrics matter? What constitutes an alert? What does the dashboard show? What logs are mandatory?
- **End-to-end tracing across domains.** A request that flows from API gateway → compute → database → storage must be traceable as a single operation. Correlation IDs cross domain boundaries.
- **The Monitoring Engineer reviews every architecture** for observability gaps. "How would we diagnose a problem here?" must have an answer for every component.

### Shared Decision Records

Decisions made without documentation are decisions that will be relitigated.

- **Architecture Decision Records (ADRs)** for every significant cross-domain decision. Why was this option chosen? What alternatives were considered? What are the trade-offs?
- **ADRs are accessible to all specialists.** Stored in a shared repository, not in someone's personal notes. Every specialist can see the reasoning behind decisions in adjacent domains.
- **ADRs reference the WAF pillar trade-offs** they represent. "We chose Standard tier (cost) over Premium tier (performance) because this workload's SLA tolerates P99 latency up to 500ms."

### Regular Sync Cadence

Specialists must share what's changing in their domain before changes create surprises.

- **Weekly or biweekly cross-domain sync.** Each specialist shares: what changed, what's planned, and what might affect other domains.
- **Change announcements before implementation.** "We're switching to private endpoints next sprint" gives other specialists time to prepare, not react.
- **Incident reviews include cross-domain analysis.** When something breaks, trace the failure across all domains it touched. Single-domain RCAs miss systemic problems.

### Skills Matrix

Know which specialist to engage for which questions. Not every question needs a specialist, and not every specialist is needed for every question.

- **Maintain a skills matrix** mapping question types to specialists. "Cosmos DB partition key design" → Database Specialist. "Cross-region data replication affecting network costs" → Database + Network + Cost specialists.
- **Define complexity thresholds.** Simple questions (well-documented, single-domain, low-risk) don't need specialist engagement. Complex questions (novel, cross-domain, production-critical) do.
- **Cross-train where domains overlap.** The compute specialist should understand basic networking. The database specialist should understand basic identity. Shared baseline knowledge reduces unnecessary escalations.

### Standardize Interface Contracts

When specialists hand off work, the handoff must be explicit, not assumed.

- **Define interface contracts** at every domain boundary. What does the network team provide to compute? IP ranges, DNS names, NSG rules, and expected latency. What does compute provide to the database team? Connection patterns, expected throughput, identity principals.
- **Version the contracts.** When a domain changes its interface, bump the version. Downstream consumers review before accepting.
- **Test the interfaces.** Integration tests that validate cross-domain connectivity, authentication, and throughput should run in CI. Don't wait for production to find out the interface broke.
