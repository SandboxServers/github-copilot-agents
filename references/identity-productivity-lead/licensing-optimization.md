## Licensing Optimization

### The SKU Landscape

| SKU | Key Capabilities | When to Use |
|-----|-----------------|-------------|
| **M365 E3** | Office apps, Exchange, SharePoint, Teams, Entra P1, Intune, basic Purview (retention, basic DLP), basic Defender | Standard enterprise workforce — covers most collaboration and basic security needs |
| **M365 E5** | Everything in E3 + Entra P2, advanced Purview (insider risk, advanced DLP, eDiscovery premium), Defender for Office/Endpoint/Identity/Cloud Apps, Power BI Pro, phone system | Organizations needing advanced security, compliance, or analytics |
| **M365 E5 Security** | E3 + advanced security only (Defender suite, Entra P2) | E3 customers who need security without full E5 compliance |
| **M365 E5 Compliance** | E3 + advanced compliance only (Purview premium, insider risk, advanced eDiscovery) | E3 customers who need compliance without full E5 security |
| **M365 Business Premium** | SMB equivalent — up to 300 users, includes Intune, Defender for Business, Entra P1 | Small businesses needing security + management |
| **EMS E3/E5** | Entra, Intune, Information Protection — the identity + device management stack | Organizations on Office 365 (not M365) who need identity and device management |

### The Licensing Optimization Game

Key decisions the lead must evaluate:

1. **E3 + add-ons vs E5**: Calculate the break-even point. If you need 3+ E5-only features, E5 is usually cheaper than E3 + individual add-ons.
2. **E5 Security vs E5 Compliance vs full E5**: Pick the add-on that matches your gap. Don't buy full E5 if you only need one side.
3. **Per-user vs organization-wide**: Some features (Purview, Defender) have per-user licensing. Not every user needs E5 — segment by role and risk profile.
4. **Bundled vs standalone**: Teams Phone System is in E5. If you're buying it standalone for E3 users, check if the upgrade to E5 is cheaper.
5. **EEA/Switzerland**: Teams is unbundled from M365 in the EEA. Factor Teams Enterprise as a separate SKU.
6. **Power Platform licensing**: M365 includes seeded Power Apps/Automate rights. Know what's included vs what needs premium licensing.
7. **Multi-Geo**: Add-on requiring 5% minimum coverage of eligible users. Plan it before deployment, not after.

### License Audit Patterns
- Users with E5 licenses not using any E5-only features → downgrade candidates
- Users with standalone add-ons that would be cheaper as E5 → upgrade candidates
- Service accounts consuming user licenses → evaluate alternatives (app registrations, managed identity)
- Shared mailboxes on licensed accounts → shared mailboxes don't need licenses in most cases
- Departed users still consuming licenses → offboarding process gap
