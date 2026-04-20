## Cloud Engineering Anti-Patterns

Broad failure modes that affect cloud engineering organizations. These are structural and process problems — distinct from the engagement-specific anti-patterns in [org-anti-patterns.md](org-anti-patterns.md).

### No Clear Division of Responsibilities

When division boundaries are fuzzy, work either gets duplicated or falls through the cracks. If both Platform Engineering and DevOps think the other team owns pipeline security, nobody owns pipeline security. Every division needs a written scope of responsibility, and every engagement needs explicit ownership assignments. Ambiguity is not flexibility — it is a gap waiting to become an incident.

The most common symptom: two specialists do overlapping work, or nobody does the work because each assumes the other division owns it. This is especially dangerous for cross-cutting concerns like monitoring, tagging, and policy enforcement where ownership is genuinely shared. Shared ownership must be explicitly divided into primary and supporting roles.

### Scope Creep on Every Engagement

The customer asks for a VNet design. During the engagement, they also want DNS, firewall rules, a load balancer, and "while you're at it" a full hub-spoke topology. Without scope discipline, every engagement grows beyond the original request, timelines slip, and specialists get pulled away from other committed work. Scope changes require a formal re-sizing and re-approval, not a verbal agreement in a meeting.

Scope creep is particularly insidious because each individual addition seems small and reasonable. The cumulative effect is an engagement that takes twice as long, involves twice the specialists, and delivers half the quality. Track scope changes formally. If the engagement has grown beyond its original sizing, re-size it — do not pretend the original estimate still holds.

### No Engagement Templates

Every project starts from scratch — new documents, new folder structures, new checklists. Without standardized templates, each engagement reinvents the process, quality varies wildly, and onboarding new specialists takes longer than it should. Templates for engagement plans, architecture reviews, handoff documents, and retrospectives should exist and be used by default.

The cost of no templates is invisible until you compare: two similar engagements by different leads produce completely different artifacts, with different levels of detail, covering different aspects. Templates establish a floor for quality and completeness. They also reduce startup time — a new engagement can begin with a populated template rather than a blank page.

### Missing Mandatory Gates

Security review, cost review, documentation review — these gates exist because skipping them has caused real problems. When gates are optional or unenforced, they get skipped under timeline pressure every time. Gates must be mandatory with explicit sign-off. No exceptions for "simple" engagements — simple engagements get simple reviews, but they still get reviews.

The most dangerous variant: gates that exist on paper but are rubber-stamped in practice. A security review that always approves without substantive feedback is not a gate — it is theater. Gate reviewers must have the authority and the expectation to block work that does not meet standards. If gates never block anything, they are not working.

### Single-Threaded Expertise

One person knows the Entra ID configuration. One person understands the Terraform module library. One person can troubleshoot the networking stack. When that person is on vacation, sick, or leaves the organization, that knowledge walks out the door. Every critical domain needs at least two people who can operate independently. Single-threaded expertise is a staffing risk, not a staffing plan.

The fix is not just documentation — it is pairing. Have the backup person actively participate in the domain, not just read about it. Documented knowledge helps, but operational experience is what makes someone capable of owning a domain independently. Budget time for cross-training as a regular operational investment, not a one-time knowledge transfer.

### No Retrospective Cadence

The engagement ends, the team moves to the next project, and nobody captures what worked or what failed. Six months later, the same mistakes repeat on a different engagement with different people. Retrospectives are not optional post-mortems — they are the mechanism by which the organization learns. Without a regular cadence, institutional memory does not exist.

The excuse is always the same: "we're too busy with the next engagement." That is exactly the problem retrospectives are designed to solve. If the organization is too busy to learn from its mistakes, it will keep making them. Block retrospective time on the calendar before the engagement starts, not after it ends.

### Over-Promising Timelines Without Specialist Input

Leadership commits to a delivery date before consulting the specialists who will do the work. The specialists discover complexity that was not visible during the sales conversation, but the date is already locked. Over-promising creates pressure to skip gates, cut corners, and deliver incomplete work. Timelines must include specialist input before they become commitments.

The dynamic is predictable: leadership wants to say yes to the customer, so they commit to an optimistic timeline. The specialists inherit a deadline they did not set and cannot meet without cutting corners. The fix is structural — no timeline commitment without specialist sign-off on feasibility. This is not bureaucracy; it is how organizations avoid setting their own teams up for failure.

### Not Tracking Cross-Division Dependencies

The compute team needs the network team to finish subnet allocation before they can deploy. The DevOps team needs architecture decisions finalized before they can write IaC. When dependencies are not tracked explicitly, teams block each other without visibility into why. A shared dependency tracker — even a simple one — prevents silent bottlenecks.

The worst case: two teams are each waiting on the other and neither realizes it. Without a dependency map, circular blocks go undetected until someone asks why nothing has progressed. Map dependencies at the start of every engagement and update them as the work evolves.

### Treating All Engagements the Same

A firewall rule change and a greenfield multi-region deployment do not need the same process. When every engagement follows the same heavyweight lifecycle regardless of complexity, small engagements take too long and large engagements get rushed. Engagement sizing exists to match process weight to actual scope. Use it.

The opposite failure is equally damaging: treating large engagements like small ones because the customer wants speed. A complex multi-division engagement that skips planning and goes straight to execution will spend more time on rework than it saved by skipping the plan. Match process to complexity — in both directions.

### No Escalation Path When Specialists Disagree

The security analyst says the architecture is non-compliant. The compute engineer says the compliant option does not meet performance requirements. Without a clear escalation path, disagreements stall engagements or get resolved by whoever argues loudest. Escalation paths need to be defined before disagreements happen — not invented during them.

Escalation does not mean one person overrules another. It means the trade-off is surfaced to someone with the authority and context to make a decision. The decision should be documented as an architecture decision record — including what was decided, why, what alternatives were considered, and what risks were accepted.

### Documentation Debt

Work gets done but not captured. Architecture decisions live in Slack threads. Configuration choices exist only in the specialist's memory. Runbooks are "on the list" but never written. Documentation debt compounds — every undocumented decision makes the next engagement harder, the next handoff riskier, and the next incident longer to resolve.

The documentation gate exists specifically to prevent this. But documentation debt also accumulates between engagements — process changes, tooling updates, and organizational decisions that nobody writes down. Treat documentation as a deliverable with the same rigor as code. If it is not documented, it did not happen.

### Capacity Planning Failures

Specialists are committed to three engagements simultaneously, each expecting full-time attention. Nobody tracks allocation, so overcommitment is invisible until deadlines slip. Capacity planning is not optional overhead — it is how the organization delivers on its commitments. If a specialist is allocated to more work than they can complete, something will not get done. Better to know that upfront than to discover it at the deadline.

Capacity failures cascade. When one specialist slips, downstream teams that depend on their output also slip. A single overcommitted network engineer can delay three engagements across three divisions. Track capacity at the organizational level, not just the division level, because dependencies cross division boundaries.

### Reactive Instead of Proactive Engagement Intake

The organization takes every request as it arrives, with no intake process and no prioritization. Whoever asks first gets resources, regardless of strategic value or organizational capacity. Without intake discipline, low-value engagements consume the same resources as high-value ones, and the organization is always reacting instead of planning. An intake process with clear criteria — business impact, technical complexity, resource requirements — ensures the organization works on what matters most.

### Ignoring Technical Debt Across Engagements

Each engagement delivers a working solution but leaves behind shortcuts, workarounds, and unfinished automation. Nobody owns the debt because it spans engagements. The next team inherits it, works around it, and adds more. Over time, the accumulated technical debt makes every new engagement slower and more fragile. Track technical debt as an organizational liability and allocate capacity to address it — do not assume it will be handled "next time."
