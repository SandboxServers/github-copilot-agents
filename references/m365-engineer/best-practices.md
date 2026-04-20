# M365 Administration Best Practices

Proven practices for governing, securing, and operating Microsoft 365 at enterprise scale.

---

## License Management — Audit and Right-Size

- Conduct **quarterly license audits** — compare assigned licenses against actual feature usage via Microsoft 365 usage reports and Entra ID sign-in logs
- Right-size E3 vs E5 per role: E5 for security teams (Defender suite), compliance teams (Purview advanced), and executives (Copilot); E3 or F-series for frontline workers
- Use group-based licensing in Entra ID — assign licenses via dynamic groups based on department, job title, or location for automatic provisioning and deprovisioning
- Reclaim licenses from inactive accounts (no sign-in for 30+ days) — automate detection with a Power Automate flow or Entra ID access review
- Track add-on licenses separately (Copilot, Power BI Pro, Visio, Project) — add-ons often purchased reactively and never reclaimed

## Security Baseline — Defaults or Conditional Access

- For tenants without Entra ID P1: enable **Security Defaults** — enforces MFA for all users, blocks legacy authentication, and protects admin roles
- For tenants with Entra ID P1/P2: replace Security Defaults with **Conditional Access policies** for granular control — target by user group, application, device state, location, and risk level
- Baseline CA policies to deploy: require MFA for all users, block legacy authentication, require compliant device for Office apps, require MFA for admin roles, block access from high-risk sign-ins
- Enable Entra ID Identity Protection risk-based policies — auto-remediate risky sign-ins (require MFA) and risky users (require password change)
- Deploy Entra ID Privileged Identity Management (PIM) — just-in-time admin role activation with approval workflows and audit trails

## Data Governance — Retention, Labels, DLP from Day One

- Deploy **retention policies** across Exchange, SharePoint, OneDrive, and Teams before users generate content — retrofitting retention is complex and audit-risky
- Publish **sensitivity labels** with clear naming (Public, Internal, Confidential, Highly Confidential) — start with manual labeling, then layer auto-labeling policies after tuning
- Configure **DLP policies** in test mode first across Exchange, Teams, SharePoint, and endpoint — monitor for false positives for 2-4 weeks before enforcement
- Enable default sensitivity labels for new documents and emails — reduces the labeling burden on users and ensures minimum classification
- Use Purview Data Map to discover and classify data across M365 and beyond — feeds into sensitivity labeling and DLP policy decisions

## Device Management — Intune Enrollment and Compliance

- Enroll all corporate devices in **Microsoft Intune** — Windows (Autopilot), iOS/iPadOS (DEP), Android (zero-touch) for consistent policy application
- Define **compliance policies** per platform: require encryption, minimum OS version, antivirus active, screen lock configured
- Use compliance policies as **Conditional Access grant controls** — non-compliant devices cannot access Exchange, SharePoint, or Teams
- Deploy **app protection policies (MAM)** for BYOD — protect corporate data in Outlook, Teams, OneDrive without requiring device enrollment
- Configure Windows Autopilot for zero-touch deployment — pre-provisioned devices ship directly to users, join Entra ID, and receive policies at first boot

## SharePoint Governance — Hubs, Naming, Quotas

- Design a **hub site hierarchy** aligned to business units or functions — hubs provide shared navigation, search scope, and consistent branding across associated sites
- Enforce **site naming conventions** via M365 group naming policy — prefixes and suffixes prevent ambiguity (e.g., `PROJ-ContosoRedesign`, `DEPT-Marketing`)
- Set **per-site storage quotas** — prevent any single site from consuming the entire pooled tenant storage; monitor via SharePoint admin center storage reports
- Implement **site lifecycle policies** — inactive sites (no activity for 90+ days) trigger owner notifications; sites with no response are archived or deleted
- Control **external sharing** per site — tenant default set to "People in your organization"; override to broader sharing only for designated collaboration sites

## Exchange Online — Mail Authentication and Transport

- Configure **SPF, DKIM, and DMARC** for all domains — SPF limits authorized senders, DKIM signs messages, DMARC tells receivers how to handle failures
- Target DMARC policy progression: `p=none` (monitor) → `p=quarantine` → `p=reject` over 3-6 months using aggregate reports to identify legitimate senders
- Use **Defender for Office 365 preset security policies** (Standard or Strict) as the baseline — avoid building custom anti-phishing, Safe Links, and Safe Attachments policies from scratch
- Minimize **transport rules** — migrate content-based detection to Purview DLP; reserve transport rules for routing and header manipulation
- Enable **audit logging** for mailbox actions (MailItemsAccessed, Send, MoveToDeletedItems) — required for compliance and breach investigations

## Power Platform — Environment Strategy and CoE

- Create **dedicated environments** for production, development, and sandbox — isolate production workloads from maker experimentation
- Deploy **DLP policies per environment** — restrict production environments to business connectors only; allow broader connector access in sandbox
- Deploy the **CoE Starter Kit** — provides inventory of all apps, flows, connectors, and makers; enables adoption dashboards and cleanup automation
- Govern **custom connectors** — require registration and review before deployment to production; classify in DLP policies alongside standard connectors
- Enable **managed environments** for production — adds admin controls including sharing limits, solution checker enforcement, and weekly usage insights

## Monitoring — Service Health, Message Center, Usage

- Assign a **Message Center reader role** to a team member or shared mailbox — ensure someone tracks upcoming changes, feature retirements, and deprecations
- Integrate **Service Health alerts** into a Teams channel via Power Automate or the Microsoft 365 Service Health API — don't rely on users to report outages
- Review **M365 usage reports** monthly (active users, storage, adoption by workload) — identify underused licenses and adoption gaps
- Enable **audit log search** and configure alert policies for high-risk activities (admin role changes, eDiscovery searches, mailbox forwarding rules)
- Monitor **Secure Score** weekly — prioritize improvement actions by impact and difficulty; target 80%+ as a governance KPI

## Backup — Third-Party Protection for M365 Data

- Microsoft provides **infrastructure resilience** (data center redundancy, geo-replication) but not item-level backup and point-in-time restore for user data
- Deploy a **third-party backup solution** (Veeam, AvePoint, Commvault, Druva) covering Exchange, SharePoint, OneDrive, and Teams
- Define **RPO and RTO** per workload — email recovery expectations differ from SharePoint site recovery; document and test against targets
- **Test restores quarterly** — verify mailbox restore, SharePoint site restore, and Teams conversation restore actually work end-to-end
- Include **Entra ID configuration backup** — Conditional Access policies, group configurations, and app registrations are not covered by M365 backup tools by default
- Consider **Microsoft 365 Backup** (native preview service) for Exchange, OneDrive, and SharePoint — evaluate against third-party solutions for coverage gaps
- Document backup scope explicitly — Teams channel conversations, Planner tasks, and Loop components may not be covered by all backup vendors

## Identity and Access — Zero Trust Foundation

- Implement **passwordless authentication** where possible — FIDO2 keys, Windows Hello for Business, or Microsoft Authenticator phone sign-in reduce phishing risk
- Use **Entra ID access reviews** for guest accounts and privileged roles — quarterly reviews for admin roles, semi-annual for guest users
- Deploy **cross-tenant access settings** to control collaboration with partner organizations — configure inbound and outbound trust for MFA and device compliance
- Enable **continuous access evaluation (CAE)** — near-real-time enforcement of token revocation when user risk changes, rather than waiting for token expiry
- Configure **authentication strengths** in Conditional Access — require phishing-resistant MFA for admin roles and sensitive applications
