## Gotchas in Identity & Productivity Leadership

These are the non-obvious traps that catch teams even when the strategy is sound. They don't look like problems until they are.

### E3 vs E5 Licensing Feature Gates

Not all M365 features are available in every SKU. Teams that assume "we have M365" without checking the SKU discover gaps at deployment time.

- **Purview Information Protection** auto-labeling requires E5 or the Information Protection add-on. E3 gets manual labeling only.
- **Entra ID P2** (PIM, Identity Protection risk policies, access reviews) is included in E5 but not E3. E3 includes P1 only — no PIM, no risk-based Conditional Access.
- **Defender for Office 365 Plan 2** (automated investigation, attack simulation) is E5. Plan 1 (Safe Attachments, Safe Links) is E3 with limitations.
- **eDiscovery Premium** (custodian management, review sets, predictive coding) is E5. E3 gets eDiscovery Standard only.
- **Viva suite components** are licensed separately from M365 base SKUs. Viva Topics, Viva Goals, and Viva Learning premium features require add-on licenses.
- **Power BI Pro** is included in E5 but not E3. Teams relying on Power BI dashboards need separate Pro licenses or an E5 upgrade.

**Reality:** Build a feature-to-SKU matrix before designing governance policies. A Purview deployment that assumes auto-labeling will fail on an E3 tenant.

### Cross-Product Policy Interactions

Conditional Access, Intune compliance, Purview DLP, and Defender policies are configured in different admin centers but evaluate against the same users, devices, and sessions. They interact in ways the documentation doesn't make obvious.

- A Conditional Access policy requires a compliant device. Intune marks the device non-compliant due to a missed OS update. The user is blocked from all cloud apps — not just the one you were trying to protect.
- Purview DLP blocks sharing a file in Teams. The user moves to email. Exchange DLP has a different threshold. The same content is allowed via email but blocked in Teams.
- Defender for Cloud Apps session policies override Conditional Access app control settings. Conflicting rules between the two produce unpredictable enforcement.
- Session controls in Conditional Access interact with app proxy and Defender for Cloud Apps. The evaluation chain is: Conditional Access → app proxy → MCAS session policy. Miss one layer and the behavior surprises you.

**Reality:** Test cross-product scenarios end-to-end. A policy that works in one admin center may conflict with policies in another. Use test accounts across all products before broad deployment.

### Monthly M365 Updates and Change Fatigue

Microsoft ships updates to M365 continuously. Features appear, move, rename, or change behavior monthly. Users experience this as instability. IT experiences this as perpetual catch-up.

- **Feature rollouts via Message Center** give 30-60 days notice, but many orgs don't monitor Message Center. Features arrive as surprises.
- **Targeted release rings** let you preview changes — but only if you've configured them and have staff assigned to validate.
- **UI changes in Teams, Outlook, and SharePoint** trigger helpdesk spikes. Users report "Outlook is broken" when the ribbon changes.
- **Deprecation notices** are easy to miss. A deprecated API or feature that your automation depends on stops working with minimal warning.

**Reality:** Assign someone to triage Message Center weekly. Use targeted release for IT staff. Communicate user-impacting changes proactively. Build a change calendar visible to helpdesk and end users.

### Cross-Product DLP Licensing Dependencies

Purview DLP across M365 is not uniformly licensed. Teams DLP in particular has licensing requirements beyond the base M365 SKU.

- **Teams DLP** (chat and channel messages) requires E5 Compliance or the DLP add-on. E3 + E5 Compliance add-on is the minimum.
- **Endpoint DLP** requires E5 or A5 licensing. Onboarding devices to endpoint DLP also requires Defender for Endpoint.
- **DLP for Power BI** requires E5. DLP policies that reference Power BI content types silently skip enforcement on E3 tenants.
- **DLP policy tips** behave differently across clients. Outlook desktop shows tips inline. Outlook mobile may not. Teams shows tips only for certain policy conditions.

**Reality:** Map every DLP scenario to its licensing prerequisite. A "unified DLP strategy" that spans Exchange, Teams, SharePoint, endpoints, and Power BI requires E5 or a stack of add-ons. Budget accordingly.

### Global Admin vs Targeted Admin Roles

Entra ID has 80+ built-in roles. Most orgs use three: Global Admin, User Admin, and "just make them Global Admin so it works."

- **Global Admin count should be 2-5.** More than that and you've lost least-privilege. Fewer than 2 and you risk lockout.
- **Exchange Admin cannot manage SharePoint.** SharePoint Admin cannot manage Teams. Teams Admin cannot manage Exchange. This is by design — but teams that expect one admin to manage "all of M365" hit permission walls.
- **Privileged Role Administrator** can assign any role, including Global Admin. This role is effectively equivalent to Global Admin and should be treated with equal caution.
- **Custom roles in Entra ID** require P1 or P2 licensing. P1 supports custom roles for app registrations only. P2 unlocks broader custom role scoping.
- **PIM (Privileged Identity Management)** requires P2. Without PIM, admin roles are permanently assigned — no time-bound activation, no approval workflows.

**Reality:** Map admin responsibilities to the narrowest built-in role that covers them. Use PIM for all privileged roles. Audit role assignments quarterly. Treat Privileged Role Administrator as Global Admin equivalent.

### Multi-Tenant Environments

Organizations with multiple Entra ID tenants face a fundamentally different challenge: tenant boundaries are hard security boundaries, not organizational boundaries.

- **B2B collaboration** works for guest access but is not seamless. Guests see a limited GAL, can't use some Teams features, and authentication switches are confusing.
- **Cross-tenant access settings** control B2B direct connect and trust settings. Misconfiguration either blocks legitimate collaboration or over-trusts external tenants.
- **Licensing does not transfer across tenants.** A user with E5 in Tenant A is unlicensed in Tenant B. Guest users need their own licensing for premium features.
- **Conditional Access policies are per-tenant.** A strong policy set in Tenant A doesn't protect Tenant B. Each tenant must be independently secured.
- **Cross-tenant synchronization** (Entra ID) can sync identities but not licenses, policies, or configurations. It reduces identity management overhead but doesn't unify governance.

**Reality:** Multi-tenant is an architectural decision with permanent governance cost. Consolidate to a single tenant when possible. When multiple tenants are required, treat each as an independent security boundary with its own full policy stack.
