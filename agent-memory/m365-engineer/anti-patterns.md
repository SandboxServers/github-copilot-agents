# M365 Administration Anti-Patterns

Common Microsoft 365 administration misconfigurations that lead to security gaps, data sprawl, wasted spend, and ungoverned environments.

---

## No Data Governance Strategy

- Tenant deployed with zero retention policies — deleted content is permanently lost after recycle bin expiration
- No sensitivity labels applied to documents or containers — all content treated identically regardless of classification
- No DLP policies active — sensitive data (PII, PHI, financial records) flows freely via email, Teams, and SharePoint sharing
- Users manually classify nothing — without auto-labeling or default labels, adoption is near zero
- **Fix:** Deploy Microsoft Purview retention policies, sensitivity labels with auto-labeling, and DLP policies from day one — data governance is exponentially harder to retrofit

## Everyone Has Global Admin

- Multiple administrators assigned Global Administrator instead of scoped roles like Exchange Admin, SharePoint Admin, or Security Admin
- Global Admin can modify any tenant setting, export any data, and elevate to any workload — massive blast radius if compromised
- No PIM (Privileged Identity Management) — admins have standing privileges 24/7 instead of just-in-time activation
- Break-glass accounts not configured — if MFA provider fails, no emergency access path exists
- **Fix:** Assign least-privilege workload-specific roles; enforce PIM with time-bound activation; maintain two cloud-only break-glass accounts excluded from Conditional Access

## Not Using Intune for Device Management

- Relying solely on on-premises Group Policy (GPO) for device management — no control over remote or BYOD devices
- Devices accessing M365 data without compliance checks — no way to verify encryption, OS patch level, or antivirus status
- App protection policies not deployed — corporate data in Outlook, Teams, and OneDrive on unmanaged mobile devices has no wipe capability
- **Fix:** Enroll corporate devices in Intune; deploy compliance policies as Conditional Access grant controls; use app protection policies (MAM) for BYOD scenarios

## Exchange Transport Rules Instead of DLP

- Complex chains of Exchange transport rules used to detect sensitive content — brittle regex patterns, no built-in sensitive info types
- Transport rules only cover email — Teams chat, SharePoint sharing, and OneDrive sync are completely unmonitored
- No incident reports or user notifications — violations silently blocked or logged in a mailbox nobody monitors
- **Fix:** Migrate to Microsoft Purview DLP policies that cover Exchange, Teams, SharePoint, OneDrive, and endpoint in a unified framework

## No License Optimization

- All users assigned E5 licenses regardless of role — frontline workers, conference rooms, and shared mailboxes on full E5
- Paid for Defender for Endpoint, Phone System, and Power BI Pro but features never deployed or configured
- No regular license audit — departed users retain licenses for months; service accounts consume full user licenses
- **Fix:** Conduct quarterly license audits; right-size E3 vs E5 per role; use F-series licenses for frontline; reclaim licenses from inactive accounts within 30 days

## SharePoint Sprawl Without Hub Site Governance

- Hundreds of standalone SharePoint sites with no hub site association — no shared navigation, search scope, or consistent branding
- No site creation policy — users create Communication and Team sites ad hoc with no naming convention or lifecycle
- Storage consumed unevenly with no quotas — one team's video library exhausts pooled tenant storage
- **Fix:** Design a hub site hierarchy aligned to business units; restrict site creation to approved processes; set per-site storage quotas; enforce naming conventions

## Not Blocking Legacy Authentication

- Legacy authentication protocols (POP3, IMAP4, SMTP AUTH, older ActiveSync) bypass MFA entirely
- Attackers use password spray against legacy endpoints — Conditional Access MFA policies are never evaluated
- Authentication policies and Security Defaults both block legacy auth, but neither has been enabled
- **Fix:** Enable Security Defaults (simplest) or create a Conditional Access policy blocking legacy authentication for all users; monitor sign-in logs for legacy protocol usage before enforcing

## Power Platform Ungoverned

- No DLP policies — Power Automate flows connect business data connectors (SharePoint, Dataverse) to consumer connectors (Twitter, Dropbox) without restriction
- All users build in the default environment — production and experimental flows share the same Dataverse instance
- No environment strategy — no separation between dev, test, and production; no solution-aware development
- CoE Starter Kit not deployed — no visibility into connector usage, flow inventory, or maker activity
- **Fix:** Create DLP policies per environment; provision dedicated environments for production workloads; deploy the CoE Starter Kit for inventory and governance dashboards

## Manual User Provisioning

- Users created manually in Microsoft 365 admin center — no automation, inconsistent attribute population
- Group memberships assigned individually — no dynamic groups based on department, location, or role
- Offboarding is ad hoc — accounts disabled days or weeks after departure; licenses not reclaimed promptly
- **Fix:** Implement automated identity lifecycle with Entra ID provisioning from HR source (Workday, SAP, CSV); use dynamic groups for license and policy assignment; automate offboarding with lifecycle workflows

## Not Monitoring Service Health Proactively

- Service Health dashboard and Message Center never checked — outages discovered when users report them
- No integration with alerting tools — Service Health API not connected to Teams channel, email, or ITSM
- Message Center posts about feature retirements and breaking changes missed — surprise disruptions at rollout
- **Fix:** Subscribe to Service Health email notifications; integrate Message Center with a dedicated Teams channel via Power Automate; assign a Message Center reader role to track change communications

## No Backup Strategy for M365 Data

- Assumption that Microsoft handles all backup and recovery — Microsoft provides infrastructure resilience, not item-level backup
- Retention policies are compliance tools, not backup — they don't protect against accidental bulk deletion followed by recycle bin purge
- No third-party backup for Exchange mailboxes, SharePoint sites, OneDrive files, or Teams conversations
- **Fix:** Deploy a third-party M365 backup solution (Veeam, AvePoint, Commvault) covering all workloads; test restore procedures quarterly; document RPO/RTO expectations

## Default Sharing Settings Too Permissive

- SharePoint and OneDrive sharing set to "Anyone with the link" — anonymous access links created by default
- External sharing enabled at the tenant level with no per-site overrides — confidential project sites share with the same permissiveness as public sites
- No expiration on sharing links — links created years ago still grant access to sensitive documents
- **Fix:** Set tenant default to "People in your organization"; override per-site for collaboration sites; enforce link expiration (30-90 days); require re-authentication for external guests
