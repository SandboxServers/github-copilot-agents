## Identity Is the Foundation

Nothing works if identity is broken. Every M365 service authenticates through Entra ID, so identity architecture decisions affect every workload.

### The Identity–Everything Connection

| Identity Decision | What It Affects |
|-------------------|----------------|
| **Conditional Access policies** | Who can access Teams, SharePoint, Exchange, Power Platform, and from what devices/locations/risk levels |
| **MFA enforcement** | Every user login to every M365 service — balance security with friction |
| **Device compliance (Intune)** | Conditional Access uses Intune compliance signals — unmanaged devices get blocked or limited |
| **Managed identity & service principals** | Automation, integrations, app registrations — the non-human identity layer |
| **PIM (Privileged Identity Management)** | Just-in-time admin access to Exchange, SharePoint, Teams admin roles |
| **B2B guest access** | External collaboration in Teams, SharePoint sharing — identity governs the boundaries |
| **Conditional Access + App Protection** | Intune app protection policies enforced via Conditional Access — the zero trust enforcement loop |

### Zero Trust — The Operating Model

Zero Trust is not a product. It is a principle set that the Identity & Productivity Lead implements across M365:

1. **Verify explicitly**: Conditional Access evaluates every access request — user, device, location, risk, app
2. **Least privilege**: PIM for admin roles, RBAC for SharePoint/Teams, scoped app permissions
3. **Assume breach**: Continuous access evaluation, session controls, Defender for Identity signals

The three-tier protection model from Microsoft:
- **Starting point**: MFA for all, block legacy auth, basic Conditional Access
- **Enterprise**: Risk-based policies (sign-in risk, user risk), compliant device requirements, app protection
- **Specialized security**: Managed device only, session restrictions, strict Conditional Access

### The Intune–Entra Partnership

Intune and Entra work as a feedback loop:
1. Intune defines what a "compliant device" means (OS version, encryption, antivirus, jailbreak detection)
2. Entra Conditional Access uses "require compliant device" as a grant control
3. Intune app protection policies protect data on unmanaged devices
4. Conditional Access requires app protection for mobile access
5. Defender for Endpoint feeds risk signals into Intune compliance → Entra Conditional Access

This is the zero trust device enforcement chain. Break any link and the chain fails.
