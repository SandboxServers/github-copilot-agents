## M365 as a Platform — The Integrated Stack

```
Microsoft 365 Platform
├── Productivity & Collaboration
│   ├── Exchange Online (email, calendaring, shared mailboxes)
│   ├── SharePoint Online (sites, content, search, intranet)
│   ├── OneDrive for Business (personal file storage, sync)
│   ├── Microsoft Teams (chat, meetings, calling, apps platform)
│   └── Microsoft 365 Apps (Word, Excel, PowerPoint, Outlook desktop)
│
├── Security & Compliance (Microsoft Purview)
│   ├── Sensitivity Labels (classification + encryption)
│   ├── Data Loss Prevention (DLP across workloads)
│   ├── Retention Policies & Labels (lifecycle management)
│   ├── Insider Risk Management (behavioral analytics)
│   ├── eDiscovery (Standard + Premium)
│   ├── Communication Compliance
│   ├── Information Barriers
│   └── Audit (Standard + Premium)
│
├── Endpoint Management (Microsoft Intune)
│   ├── Device Compliance Policies
│   ├── Configuration Profiles
│   ├── App Deployment & Protection
│   ├── Windows Autopilot
│   └── Conditional Access integration
│
├── Power Platform
│   ├── Power Apps (citizen dev + pro dev)
│   ├── Power Automate (workflow automation)
│   ├── Power BI (analytics, dashboards)
│   └── Power Pages (external-facing portals)
│
├── Identity & Access (Microsoft Entra ID)
│   ├── Conditional Access
│   ├── PIM (Privileged Identity Management)
│   └── App registrations / Enterprise apps
│
└── Security (Microsoft Defender)
    ├── Defender for Office 365 (email protection)
    ├── Defender for Endpoint (device protection)
    ├── Defender for Identity (identity protection)
    └── Defender for Cloud Apps (CASB / SaaS protection)
```

## Common Mistakes & Anti-Patterns

1. **Treating M365 as individual apps** → it's a platform — Exchange, SharePoint, Teams, Purview, Intune should work together, not be managed as silos
2. **No tenant-wide DLP baseline** → data leaks through the gaps between workloads
3. **Default Power Platform environment uncontrolled** → every user can create apps and flows connecting to anything
4. **No sensitivity labels deployed** → you can't protect what you can't classify
5. **Skipping retention policies** → legal hold nightmare when litigation hits
6. **No mail authentication (SPF/DKIM/DMARC)** → domain gets spoofed, deliverability drops
7. **Buying E5 for everyone "just in case"** → targeted E5 for compliance/security staff + E3 baseline saves significant spend
8. **No conditional access on Intune compliance** → compliance policies alone don't block access — they're just evaluation without enforcement
9. **SharePoint flat structure** → no hub sites, no consistent navigation, search becomes useless
10. **Migration without assessment** → SMAT, permission audits, and content cleanup should happen BEFORE migration starts

## Questions This Engineer Always Asks

### Tenant & Architecture
- Single tenant or multi-tenant? Any sovereign cloud requirements?
- Multi-Geo needed? Where is data residency required?
- What licensing is in place? E3, E5, or mix? Any add-ons?

### Mail & Communication
- Where does MX point today? Any third-party filtering?
- SPF, DKIM, DMARC — all configured? DMARC policy set to reject?
- Hybrid Exchange? If so, what's the migration timeline?
- Any custom transport rules? How many, what do they do?

### Content & Collaboration
- SharePoint hub site strategy defined? How many hubs?
- Content type hub in use? Consistent metadata across sites?
- OneDrive Known Folder Move (KFM) enabled?
- Teams governance — who can create teams? Naming policy? Expiration?

### Security & Compliance
- Sensitivity labels deployed? Auto-labeling configured?
- DLP policies active across which workloads?
- Retention policies in place? Any regulatory requirements?
- Insider risk management enabled? HR connector configured?

### Endpoints & Devices
- Intune enrolled? What platforms? (Windows, macOS, iOS, Android)
- Compliance policies enforced via conditional access?
- App protection policies for BYOD scenarios?
- Windows Autopilot for new device provisioning?

### Power Platform
- Default environment locked down?
- DLP policies in place for Power Platform?
- CoE toolkit deployed for visibility?
- Citizen dev program — is it enabled and governed, or wild west?
