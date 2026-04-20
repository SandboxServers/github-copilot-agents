## Organization Anti-Patterns

These are the failure modes that undermine engagement quality. Every one of these has happened, and every one of them is avoidable.

### Skipping Security Review

Deploying to production without security sign-off because "we're behind schedule." Security review is not a nice-to-have — it has veto power for a reason. Skipping security creates vulnerabilities that are harder and more expensive to fix after deployment than during design. If the timeline doesn't accommodate security review, the timeline is wrong.

### No Cost Estimate Before Deploying

Provisioning Azure resources without a cost projection because "we'll optimize later." Later never comes, and the first bill is always a surprise. The cost analyst reviews every new Azure spend before it happens. Architecture decisions made without cost input produce architectures that customers cannot afford to run.

### No Documentation Deliverables

Closing an engagement without architecture docs, runbooks, or decision records because "the team knows how it works." The team that built it is not the team that operates it. Undocumented systems become unmaintainable systems. The documentation gate exists because every engagement that skipped documentation created operational problems within months.

### Specialists Working in Silos

A network engineer designs the VNet without talking to the compute engineer who needs specific subnet sizing. A database specialist selects a service without consulting the security analyst on compliance requirements. Silos produce architectures where the pieces don't fit together. Cross-division coordination is not overhead — it is how the organization produces coherent solutions.

### No Retrospective

Finishing an engagement and moving straight to the next one without reviewing what worked and what didn't. Without retrospectives, the same mistakes repeat across engagements. The retrospective agent runs a structured review after every engagement — capturing lessons learned, process improvements, and action items that feed back into how the organization operates.

### Over-Engineering for Small Engagements

Assigning all four divisions and running a full five-phase lifecycle for a simple firewall rule change or a single pipeline buildout. Small engagements need 1-2 specialists and can be completed in 1-2 weeks. Match the process to the scope — mandatory gates still apply, but the engagement doesn't need a full planning phase with all division leads.

### Under-Sizing Complex Engagements

Scoping a greenfield multi-region HIPAA-compliant deployment as a medium engagement because the customer wants it done in three weeks. Under-sizing produces skipped gates, incomplete documentation, untested failover, and technical debt that surfaces in production. If the scope is large, size it as large and negotiate timeline expectations honestly.

### Ignoring Dependency Chains

Starting compute deployment before the network topology is finalized. Writing IaC before architecture decisions are made. Configuring M365 before the Entra ID foundation is established. Ignoring dependencies creates rework — work that has to be torn down and rebuilt when the upstream decision changes. The dependency flow exists for a reason: identity → platform → specialty → DevOps.
