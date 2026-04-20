## Anti-Patterns in Identity & Productivity Leadership

These anti-patterns emerge when the Identity & Productivity Lead treats identity, collaboration, and governance as separate concerns instead of a unified strategy.

### Security Maximalism at Productivity's Expense

When MFA fires every 15 minutes and Conditional Access blocks legitimate workflows, people find workarounds. The most secure system nobody can use isn't secure; it's abandoned.

- Phishing-resistant MFA on every session including compliant managed devices. Users switch to personal devices.
- Blocking all external sharing in SharePoint. Teams email attachments or use Dropbox — ungoverned.

**Fix:** Risk-proportional controls. Compliant devices get longer sessions. Low-risk apps get less friction. High-risk actions get step-up auth.

### No Unified Identity Strategy

Entra ID for M365. Different IdP for legacy apps. SaaS apps with local accounts. Untracked service principals. No single view of access, no consistent MFA, no way to fully offboard.

- Password fatigue from different credentials. Offboarding misses third-party accounts.

**Fix:** Entra ID is the single identity plane. Every app authenticates through Entra. No exceptions without a documented, time-bound waiver.

### Tool Sprawl Without Consolidation

Teams for chat. Slack for engineering. Yammer for announcements. SharePoint, OneDrive, and shared mailboxes all have documents. Nobody knows where anything is.

- Users waste time searching across platforms. IT maintains policies across multiple admin surfaces.

**Fix:** Pick a primary collaboration stack. Consolidate. M365 is the platform — Teams, SharePoint, OneDrive as governed defaults. Exceptions require justification.

### Not Measuring User Adoption

Deployed Teams telephony, Power Automate, and Planner. Nobody checks whether anyone uses them. Licenses active, bills paid, usage near zero. "We deployed it" is not "people use it."

**Fix:** Define adoption targets before deployment. Track via M365 Usage Reports. Measure active users, not licensed users.

### M365 Feature Deployment Without Change Management

Microsoft ships monthly updates. New features appear without warning. Users panic, file tickets, or ignore useful capabilities.

- Teams meeting experience changes. Users think something is broken.
- Sensitivity labels appear in Outlook because Purview was configured server-side. Users don't know what they mean.

**Fix:** Subscribe to M365 Message Center. Triage monthly. Communicate impactful changes before they land. Use targeted release for validation.

### Conditional Access Complexity Spiral

Twenty-seven CA policies, built incrementally over two years by three admins. Some overlap. Some conflict. Nobody can explain what happens when a contractor on an unmanaged device accesses SharePoint from a foreign IP.

- Troubleshooting requires mentally simulating evaluation order across all policies.
- New policies added without auditing existing ones. Redundant rules accumulate.

**Fix:** Fewer policies with broader scope. Document every policy's purpose. Review quarterly. Test with "What If" before deploying.

### No License Optimization

Everyone gets E5 "just in case." Nobody tracks which features are used. The CFO asks why M365 costs $2M/year.

- Users with E5 who only use Outlook and Teams. Add-on licenses for capabilities already in the base SKU.
- Departed users retaining licenses for months because deprovisioning doesn't trigger reclamation.

**Fix:** Quarterly license audits. Map features to SKUs. Downgrade users who don't need premium. Automate license reclamation on offboarding.

### Shadow IT From Restrictive Policies

Block everything. Users route around. Now you have unauthorized SaaS apps, personal cloud storage, and unapproved AI tools — all invisible to IT.

- Blocking Power Platform drives citizen developers to external low-code tools with zero governance.
- Strict 6-week app approval processes push teams to sign up for SaaS tools with a credit card.

**Fix:** Provide governed alternatives. Make the approved path easier than the workaround. Use Defender for Cloud Apps to discover shadow IT, then rationalize.

### Intune Compliance Without Communication

Compliance policies that block non-compliant devices with zero warning. Users lose access mid-meeting because a missed OS update triggered enforcement overnight.

- Grace periods too short. No self-service remediation path.

**Fix:** Communicate requirements before enforcement. Provide grace periods. Build self-service remediation via Company Portal. Escalate gradually: notify, restrict, then block.

### No Power Platform Governance

Power Platform is either wide open (anyone builds anything, connects to anything) or completely locked down. Neither works.

- No DLP policies: a flow sends customer data to personal Gmail. Environment sprawl with no inventory or lifecycle management.

**Fix:** Center of Excellence. DLP policies per environment. Training for premium connectors. Inventory and lifecycle-manage environments.

### Identity as Infrastructure Instead of Security Perimeter

Treating Entra ID as "just the directory" — IT plumbing. In reality, identity is the primary security perimeter in a cloud-first world.

- Identity team reports to IT operations, not security. No integration between identity signals and threat detection.
- Privileged access via static group membership, not PIM with time-bound activation.

**Fix:** Identity is security. Integrate Entra ID Protection with Sentinel. Use PIM for all privileged roles. Review identity posture as part of security operations.

### Purview Policies Not Connected Across Workloads

DLP in Exchange. Different DLP in SharePoint. No DLP in Teams. Sensitivity labels configured but not uniformly enforced.

- User emails a sensitive doc — DLP catches it. Same doc shared via Teams chat — DLP doesn't apply.
- Retention policies vary by workload. Exchange 7 years, SharePoint 3, Teams none.

**Fix:** Unified Purview policies across all M365 workloads. Same DLP rules everywhere. Consistent label enforcement. Test cross-workload scenarios.
