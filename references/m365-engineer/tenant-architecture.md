## Tenant Architecture

### Single vs Multi-Tenant

| Scenario | Recommendation | Rationale |
|----------|---------------|-----------|
| Single org, one country | Single tenant | Simplest administration, unified GAL |
| Single org, global presence | Single tenant + Multi-Geo | One tenant, data residency per region |
| M&A — temporary separation | Multi-tenant + cross-tenant access | Separate until integration, B2B for collaboration |
| Regulated subsidiaries | Multi-tenant | Data isolation required by regulation |
| Sovereign cloud requirements | Separate tenant in sovereign cloud | GCC/GCC High/DoD, 21Vianet |
| Planned divestiture | Multi-tenant from day one | Cleaner separation later |

### Multi-Geo for Data Residency
- Satellite geo locations for Exchange, SharePoint, OneDrive, Teams
- Requires add-on license — minimum 5% of total seats or 250 seats
- Supported geos vary by service — verify current list in Microsoft documentation
- Search is unified across geos with security-trimmed results
- eDiscovery works across geos but may require separate searches
- Preferred Data Location (PDL) set per user controls mailbox and OneDrive placement

### Sovereign Clouds

| Cloud | Use Case | Key Differences |
|-------|----------|----------------|
| Commercial | Standard enterprise | Full feature set, global data centers |
| GCC | US government (moderate) | US data centers, FedRAMP High |
| GCC High | US government (high) | DoD-level controls, ITAR/EAR data |
| DoD | US Department of Defense | Most restrictive, DoD-only data centers |
| 21Vianet | China operations | Operated by Chinese partner, separate identity system |

### Tenant Settings
- **Organization profile**: company name, address, technical and privacy contacts
- **Release preferences**: Targeted Release (early features for select users) vs Standard Release
- **Custom domains**: verified via DNS TXT record, multiple domains supported per tenant
- **Theme and branding**: company logo, colors, sign-in page customization via Entra ID branding

### Tenant Consolidation and Migration
- Cross-tenant mailbox migration for Exchange Online
- Cross-tenant OneDrive migration via Microsoft tooling
- SharePoint cross-tenant migration (site-level)
- Requires Entra ID cross-tenant access policies on both source and target
- Plan for: license reconciliation, DLP policy alignment, Conditional Access harmonization
- Identity sync: decide between guest accounts vs migrated accounts

### Admin Centers Overview

| Admin Center | URL Pattern | Primary Scope |
|-------------|-------------|---------------|
| Microsoft 365 | admin.microsoft.com | Users, licenses, org settings |
| Exchange | admin.exchange.microsoft.com | Mail flow, mailboxes, groups |
| SharePoint | (tenant)-admin.sharepoint.com | Sites, sharing, storage |
| Teams | admin.teams.microsoft.com | Teams policies, devices, calling |
| Intune | intune.microsoft.com | Devices, compliance, apps |
| Security | security.microsoft.com | Defender, incidents, threat analytics |
| Compliance | compliance.microsoft.com | Purview, DLP, retention, eDiscovery |
| Entra | entra.microsoft.com | Identity, Conditional Access, PIM |
| Power Platform | admin.powerplatform.microsoft.com | Environments, DLP, capacity |
