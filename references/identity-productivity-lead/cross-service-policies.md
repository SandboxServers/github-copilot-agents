## M365 as an Integrated Platform

### Cross-Service Policy Integration

Purview policies don't respect service boundaries — that's the point. The lead must understand how policies span services:

| Policy Type | Services It Spans | What It Does |
|------------|-------------------|--------------|
| **Retention policies** | Exchange, SharePoint, OneDrive, Teams, Viva Engage, M365 Groups | Retain or delete content by container or across the entire tenant |
| **DLP policies** | Exchange, SharePoint, OneDrive, Teams, Endpoints, Power BI | Detect and protect sensitive information across workloads |
| **Sensitivity labels** | Office docs, emails, Teams meetings, SharePoint sites, M365 Groups, Power BI | Classify and protect content with encryption, visual markings, access controls |
| **Communication compliance** | Teams, Exchange, Viva Engage | Monitor communications for policy violations |
| **eDiscovery** | Exchange, SharePoint, OneDrive, Teams | Search and hold content across all workloads for legal/compliance |
| **Information barriers** | Teams, SharePoint, OneDrive | Prevent communication between specific groups (ethical walls) |
| **Insider risk management** | All M365 services + endpoints | Detect and investigate risky user behavior patterns |

### How Services Connect

- **Teams + SharePoint**: Every Teams channel has a SharePoint site. Teams file storage IS SharePoint. SharePoint governance = Teams file governance.
- **Teams + Exchange**: Teams calendar is Exchange calendar. Teams voicemail is Exchange. Meeting recordings flow through OneDrive/SharePoint.
- **Entra + Teams**: Conditional Access controls Teams access. Guest access policies in Entra govern Teams external collaboration.
- **Entra + SharePoint**: External sharing settings interact with Entra B2B policies. Conditional Access can scope to SharePoint specifically.
- **Purview + Everything**: Sensitivity labels, retention, DLP span all workloads. Policy inconsistency across workloads creates gaps.
- **Power Platform + M365**: Dataverse environments use Entra for auth. DLP policies in Power Platform control connector usage. Power Platform governance is part of the M365 governance story.
