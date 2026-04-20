## M365 as an Integrated Platform

### The Platform View

Microsoft 365 is not a collection of apps — it is a platform built on three pillars:

- **Identity** (Entra ID): Every service authenticates through one identity provider. One Conditional Access policy set governs all access.
- **Collaboration** (Exchange + SharePoint + Teams + OneDrive): Interconnected services that share storage, calendar, and communication infrastructure.
- **Governance** (Purview + Intune + Defender): Cross-service policies for data protection, device management, and threat defense.

The lead's job is ensuring these pillars work together. Decisions in identity ripple through collaboration. Governance constrains both. Licensing determines what's available.

### How Services Connect

These connections are not optional — they exist whether you govern them or not:

| Connection | What It Means |
|-----------|--------------|
| **Teams + SharePoint** | Every Teams channel has a SharePoint site. Teams file storage IS SharePoint. SharePoint governance = Teams file governance. |
| **Teams + Exchange** | Teams calendar is Exchange calendar. Voicemail is Exchange. Meeting recordings flow through OneDrive/SharePoint. |
| **Teams + OneDrive** | Private chat files stored in sender's OneDrive. 1:1 file sharing uses OneDrive. |
| **Entra ID + all services** | Conditional Access controls access to every M365 workload. Guest policies in Entra govern Teams, SharePoint, and Power Platform external access. |
| **Purview + all services** | Sensitivity labels, retention policies, and DLP span Exchange, SharePoint, Teams, OneDrive, and Power BI. |
| **Power Platform + M365** | Dataverse uses Entra for auth. Power Automate connects to Exchange, SharePoint, Teams. DLP policies in Power Platform control connector usage. |
| **Intune + Entra** | Device compliance feeds Conditional Access. App protection policies govern data on unmanaged devices. |

### Cross-Service Governance with Purview

Purview policies intentionally ignore service boundaries:

- **Retention policies**: Apply to Exchange, SharePoint, OneDrive, Teams, Viva Engage, M365 Groups — retain or delete content uniformly
- **DLP policies**: Detect sensitive data in Exchange, SharePoint, OneDrive, Teams, endpoints, Power BI — consistent protection everywhere
- **Sensitivity labels**: Classify and protect Office docs, emails, Teams meetings, SharePoint sites, M365 Groups, Power BI dashboards
- **Unified audit log**: Single log covering all M365 services — one place to search for activity across the platform

### Shared Security Through Conditional Access

One Conditional Access policy set protects the entire platform:

- Same MFA requirements whether accessing Teams, SharePoint, or Exchange
- Device compliance enforced uniformly — not per-app
- Location-based restrictions apply to all services (or scoped to specific apps when needed)
- Session controls via Defender for Cloud Apps apply to SharePoint, Exchange, and Teams simultaneously

**Anti-pattern**: Creating different security policies per workload. Users accessing SharePoint through Teams get one policy, the same content through a browser gets another. Inconsistency creates gaps.

### License Implications on Platform Capabilities

| License Level | Platform Impact |
|--------------|----------------|
| **E3** | Basic platform — Conditional Access P1, basic DLP, manual sensitivity labels, basic retention |
| **E5** | Full platform — risk-based Conditional Access, auto-labeling, advanced DLP, insider risk, full Defender suite |
| **Mixed E3/E5** | Governance gap risk — E5 policies may not apply to E3 users, creating inconsistent protection |

### The Integration Principle

When planning any M365 change, evaluate cross-service impact:

1. **New Teams policy** → Does it align with SharePoint sharing settings and Entra guest access?
2. **New retention policy** → Does it cover all workloads where the data type exists?
3. **New Conditional Access rule** → Which apps does it target? Does excluding one create a bypass?
4. **New Power Platform DLP** → Does it align with Purview DLP for the same data types?
5. **New sensitivity label** → Is it configured for all workloads (files, emails, sites, meetings)?
