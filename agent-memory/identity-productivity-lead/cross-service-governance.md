## Cross-Service Governance with Purview

### Why Purview Is the Governance Layer

Data in M365 doesn't stay in one service. An email attachment becomes a SharePoint file, gets shared in Teams, downloaded to a device, and forwarded externally. Governance must follow the data across every surface — that's Purview's role.

### Sensitivity Labels

Sensitivity labels classify and protect content wherever it goes:

| Label Target | What It Does |
|-------------|-------------|
| **Files and emails** | Encrypt content, apply watermarks/headers/footers, restrict copy/print/forward, control external sharing |
| **SharePoint sites and OneDrive** | Set site-level sharing restrictions, control guest access, enforce privacy settings |
| **Teams** | Control guest access to the team, meeting settings, shared channel policies |
| **M365 Groups** | Apply governance to the group and all connected services (Teams, SharePoint, Planner) |
| **Power BI** | Protect dashboards and reports — label persists when data is exported |
| **Meetings** | Control recording, transcription, who can present, watermarking in Teams meetings |

**Auto-labeling** (E5): Apply labels automatically based on sensitive information types — detect credit card numbers, SSNs, health records and label without user action.

### Retention Policies

Retention policies ensure content is kept as long as required and deleted when allowed:

- **Exchange**: Retain emails and calendar items for compliance periods, then delete
- **SharePoint and OneDrive**: Retain document versions, preserve deleted files in the Preservation Hold Library
- **Teams**: Retain chat messages and channel messages — stored in Exchange mailboxes and SharePoint respectively
- **M365 Groups**: Retention applies to the group mailbox and connected SharePoint site

**Retention labels vs retention policies**: Policies apply broadly (all Exchange, all SharePoint). Labels apply to specific items and can be set by users or auto-applied. Labels override policies when there's a conflict.

### DLP Policies

Data Loss Prevention detects and protects sensitive information across workloads:

- **Sensitive information types**: Built-in (300+) or custom patterns for detecting PII, financial data, health records, intellectual property
- **Policy tips**: Warn users before they share sensitive content — education at the point of action
- **Block actions**: Prevent sharing sensitive content externally, block upload to unsanctioned locations
- **Endpoint DLP**: Extend DLP to Windows and macOS devices — control copy to USB, print, upload to cloud apps
- **Adaptive protection**: Integrate with insider risk management — automatically apply stricter DLP to users flagged as higher risk

### Information Barriers

Information barriers prevent communication and collaboration between specific groups:

- **Use case**: Financial services (Chinese walls), legal firms, M&A scenarios
- **Scope**: Teams chat, Teams channels, SharePoint sites, OneDrive sharing
- **Implementation**: Define segments based on Entra attributes, create barrier policies between segments
- **Limitation**: Barriers are binary — blocked or not. They don't support conditional access between groups.

### Insider Risk Management

Detect and investigate risky user behavior patterns across M365:

- **Indicators**: Mass file downloads, unusual sharing patterns, data exfiltration to personal email, printing spikes before departure
- **Policies**: Departing employee data theft, data leaks, security policy violations
- **Privacy**: Pseudonymized by default — investigator sees "User-1234" until escalated
- **Integration**: Feeds into adaptive protection (DLP) and can trigger Purview eDiscovery cases

### Unified Audit Log

Single audit log covering all M365 services:

- **Scope**: Exchange, SharePoint, OneDrive, Teams, Power Platform, Entra ID, Purview, Defender
- **Retention**: 180 days standard (E3), 1 year default + 10 year option (E5 with advanced audit)
- **Search**: Compliance portal or PowerShell — filter by user, activity, date range, service
- **Key activities**: File access, sharing changes, permission modifications, admin actions, mailbox access, sensitivity label changes
- **Integration**: Export to Sentinel for advanced analytics, long-term retention, and automated investigation

### Governance Alignment Checklist

| Question | Governance Tool |
|----------|----------------|
| How long must we keep this data? | Retention policies and labels |
| Who can access this data? | Sensitivity labels + Conditional Access |
| Can this data leave the organization? | DLP policies + endpoint DLP |
| Can these groups communicate? | Information barriers |
| Is anyone behaving suspiciously? | Insider risk management |
| What happened to this data? | Unified audit log + eDiscovery |
