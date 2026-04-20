## M365 Licensing

### E3 vs E5 Comparison

| Feature | E3 | E5 |
|---------|----|----|
| Exchange Online | Plan 2 | Plan 2 |
| SharePoint Online | Plan 2 | Plan 2 |
| Microsoft Teams | Yes | Yes |
| Microsoft 365 Apps | Yes | Yes |
| Intune Plan 1 | Yes | Yes |
| Entra ID P1 | Yes | Yes |
| Entra ID P2 | No | Yes |
| Defender for Office 365 | Plan 1 (basic) | Plan 2 (full) |
| Defender for Endpoint | Plan 1 | Plan 2 |
| Defender for Identity | No | Yes |
| Defender for Cloud Apps | No | Yes |
| Purview DLP | Exchange only | Exchange + Endpoint + Teams |
| Service-side auto-labeling | No | Yes |
| Insider Risk Management | No | Yes |
| eDiscovery Premium | No | Yes |
| Phone System | No | Yes |
| Power BI Pro | No | Yes |

### Security Add-Ons

| Add-On | What It Adds | Typical Pairing |
|--------|-------------|----------------|
| Defender for Office 365 P2 | Full email protection, investigation, simulation | E3 |
| Defender for Endpoint P2 | Full endpoint detection and response | E3 |
| Entra ID P1 | Conditional Access, self-service password reset | F1/F3 |
| Entra ID P2 | PIM, risk-based Conditional Access, access reviews | E3 |
| E5 Security add-on | Full Defender suite + Entra P2 bundle | E3 |

### Compliance Add-Ons

| Add-On | Capabilities |
|--------|-------------|
| E5 Compliance | Full Purview suite — DLP, insider risk, eDiscovery Premium, communication compliance |
| Purview Audit Premium | Extended retention, crucial event audit |
| Information Protection P2 | Service-side auto-labeling |

### Teams Add-Ons

| Add-On | Purpose |
|--------|---------|
| Teams Phone Standard | Phone System for E3 users |
| Calling Plan (Domestic/International) | PSTN calling minutes |
| Teams Rooms Basic/Pro | Meeting room device licensing |
| Audio Conferencing | Dial-in numbers for Teams meetings |

### Power Platform Licensing
- **Per-user plans**: Power Apps per user, Power Automate per user — unlimited apps/flows
- **Per-app plans**: Power Apps per app — access to specific apps only
- **Premium connectors**: require Power Apps/Automate premium license (Dataverse, SQL, HTTP, custom)
- **Seeded use rights**: M365 licenses include limited Power Platform capabilities with standard connectors
- **Pay-as-you-go**: consumption-based billing for Power Apps and Power Automate

### Group-Based Licensing
- Assign licenses to Entra ID security groups instead of individual users
- Users inherit licenses from group membership
- Supports multiple groups — licenses are additive
- Error handling: insufficient licenses surface as assignment errors on the group
- Best practice: create groups per license SKU (e.g., SG-License-M365-E3, SG-License-M365-E5)

### License Assignment Best Practices
- Use group-based licensing over direct assignment for consistency
- Disable unused service plans within a license to reduce noise (e.g., Sway, Kaizala)
- Audit license usage quarterly — reclaim unused licenses
- Use Microsoft 365 admin center license reports or Entra ID sign-in logs

### Cost Optimization
- Mixed licensing: E5 for security and compliance roles, E3 baseline for general users
- F1/F3 for frontline workers — limited feature set at lower cost
- Evaluate E5 Security and E5 Compliance add-ons vs full E5 upgrade
- Remove Defender features from E5 if using third-party security stack
- Consolidate overlapping add-ons — E5 includes many features sold separately
