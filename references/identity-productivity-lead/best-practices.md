## Best Practices for Identity & Productivity Leadership

These practices produce an identity and collaboration environment where security enables productivity instead of fighting it.

### Entra ID as the Single Identity Plane

Every authentication — M365, Azure, SaaS, on-premises, partner access — flows through Entra ID. No exceptions without a documented, time-bound waiver.

- **Federate everything.** SaaS apps via SAML/OIDC. On-premises apps via App Proxy or Secure Hybrid Access partners. Legacy apps via header-based auth through App Proxy.
- **SSO everywhere.** Users authenticate once. Every subsequent app gets a token from Entra without prompting. SSO reduces password fatigue and eliminates credential sprawl.
- **Automate provisioning.** SCIM provisioning for SaaS apps. HR-driven provisioning for lifecycle (hire, move, leave). No manual account creation.
- **Consolidate directories.** One Entra ID tenant is the source of truth for identity. On-premises AD syncs to Entra via Entra Connect. Never the reverse.

### Zero Trust: Verify Explicitly, Least Privilege, Assume Breach

Zero Trust is the security model. Conditional Access is the enforcement engine. Every access request is evaluated against identity, device, location, and risk signals.

- **Conditional Access as the policy engine.** Require MFA for risky sign-ins. Require compliant devices for sensitive apps. Block legacy authentication entirely. Use named locations for trusted networks but never trust a network alone.
- **Device trust via Intune.** A compliant, managed device gets less friction. An unmanaged device gets restricted access (browser-only, no download). A non-compliant device gets blocked.
- **Least privilege via PIM.** No permanent admin roles. All privileged access is time-bound, approval-gated, and audited. Global Admin activations trigger alerts.
- **Assume breach via Identity Protection.** Risk-based policies auto-remediate risky sign-ins (force MFA) and risky users (force password reset). Integrate identity risk signals with Sentinel for SOC visibility.

### License Management: Audit, Optimize, Govern

Licenses are money. Unused licenses are wasted money. Misassigned licenses are compliance risk.

- **Quarterly license audits.** Compare assigned licenses to actual feature usage via M365 Admin Center usage reports. Downgrade users who don't use premium features.
- **Group-based licensing.** Assign licenses via Entra ID group membership. When a user changes roles, group membership changes, and licenses follow automatically.
- **Feature adoption tracking.** Don't just count licenses — measure which features within the SKU are used. An E5 user who only uses Outlook and Teams is an E3 user with an E5 price tag.
- **License reclamation on offboarding.** Automate license removal in the deprovisioning workflow. Licenses freed within 24 hours of account disable, not whenever someone remembers.

### User Adoption: Training, Champions, Feedback

Deployment is not adoption. A feature nobody uses is a feature that doesn't exist.

- **Champions program.** Identify power users in each department. Train them first. They become the peer support network that scales better than IT helpdesk.
- **Scenario-based training.** Don't teach "how to use Teams." Teach "how to run a project in Teams" — channels for topics, tabs for documents, Planner for tasks, meetings for standups.
- **Feedback loops.** Monthly surveys, usage analytics, helpdesk ticket analysis. If a feature has low adoption, find out why before blaming users.
- **Executive sponsorship.** Adoption campaigns without visible executive participation fail. When leadership uses Teams publicly, the organization follows.

### Governance: Clear Ownership for Every Service

Every M365 service needs an owner, a lifecycle policy, and usage boundaries. Ungoverned services become ungoverned data stores.

- **Teams governance.** Naming conventions, expiration policies, classification labels, guest access policies. Every team has an owner who reviews membership quarterly.
- **SharePoint governance.** Site creation policies, storage quotas, sharing defaults (internal-only by default, external by exception). Hub sites for organizational structure.
- **Power Platform governance.** Environment strategy (default, dev, production), DLP policies per environment, maker training requirements, Center of Excellence toolkit deployed.
- **Groups governance.** M365 Groups are the universal membership backbone for Teams, SharePoint, Planner, and Outlook. Govern groups and you govern all dependent services.

### Security Baseline: Conditional Access + Compliance + Classification

Security is not one policy — it's a chain. Identity verification, device compliance, and data classification work together or not at all.

- **Conditional Access baseline:** block legacy auth, require MFA for all users, require compliant devices for Office 365 apps, restrict unmanaged device access to browser-only with no download.
- **Intune compliance baseline:** require encryption, require minimum OS version, require threat protection (Defender for Endpoint), block jailbroken/rooted devices.
- **Data classification baseline:** define sensitivity labels (Public, Internal, Confidential, Highly Confidential). Auto-label where possible. Enforce labels on containers (Teams, SharePoint sites, M365 Groups).
- **Connect the chain:** Conditional Access checks device compliance (Intune), which requires Defender for Endpoint (threat detection), which feeds risk signals to Identity Protection (Entra). Break one link and the chain fails.

### Monitoring: Usage Reports, Security Dashboards, Service Health

You can't manage what you don't measure. You can't secure what you can't see.

- **M365 Usage Reports.** Active users per app, feature adoption trends, storage consumption, collaboration patterns. Review monthly. Share with business stakeholders.
- **Entra ID sign-in and audit logs.** Stream to Log Analytics. Build dashboards for failed sign-ins, risky sign-ins, Conditional Access policy hits, and PIM activations.
- **Defender dashboards.** Email threat trends, phishing simulation results, device compliance rates, vulnerability exposure. Review weekly with security team.
- **Service Health.** Monitor M365 Service Health dashboard. Subscribe to incident notifications. Correlate service incidents with helpdesk ticket spikes.
- **Viva Insights** (where licensed). Collaboration patterns, meeting overload, focus time trends. Use for organizational health, not individual surveillance.

### Integration: Identity + Device + Data as a Unified Chain

Identity protection, device management, and data governance are not three projects — they're one strategy with three pillars.

- **Identity verifies the user.** Entra ID + Conditional Access confirms who is accessing, from where, with what risk level.
- **Device confirms the posture.** Intune + Defender for Endpoint confirms the device is managed, compliant, and threat-free.
- **Data classification protects the content.** Purview sensitivity labels + DLP ensures that even if identity and device checks pass, the data is still protected based on its classification.
- **Automate the response chain.** A risky sign-in (Identity Protection) triggers a compliance re-check (Intune), which triggers a session restriction (Conditional Access), which applies additional DLP controls (Purview). The chain is automatic, not manual.
- **Sentinel ties it together.** Identity signals, device signals, and data signals all flow to Sentinel. Correlation rules detect attack patterns that no single product can see alone.
