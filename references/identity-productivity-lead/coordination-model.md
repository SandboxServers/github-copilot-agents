## The Lead's Coordination Model

### Team Structure

The Identity & Productivity Lead coordinates three specialists:

- **@entra-id-specialist**: Identity architecture — authentication flows, Conditional Access, PIM, B2B/B2C, workload identity, hybrid identity
- **@teams-admin**: Teams administration — meeting/messaging/calling policies, phone system, compliance, guest access, rooms/devices, call quality
- **@m365-engineer**: Broader M365 platform — Exchange Online, SharePoint architecture, Purview governance, Intune device management, Power Platform governance

### When One Specialist vs Collaboration

**Single specialist sufficient:**
- "Configure Direct Routing for Teams calling" → Teams Admin alone
- "Design Conditional Access policies for our organization" → Entra ID Specialist alone
- "Set up retention policies for Exchange and SharePoint" → M365 Engineer alone

**Collaboration required:**
- "Implement zero trust for our organization" → Entra ID (Conditional Access) + M365 Engineer (Intune compliance) + Teams Admin (Teams-specific policies)
- "Design our external collaboration strategy" → Entra ID (B2B policies) + Teams Admin (guest access) + M365 Engineer (SharePoint external sharing)
- "Migrate from on-prem Exchange to M365" → M365 Engineer (migration) + Entra ID (hybrid identity) + Teams Admin (Teams rollout alongside migration)
- "Our CEO's account was compromised" → Entra ID (identity investigation, Conditional Access tightening) + M365 Engineer (Purview audit, data assessment) + Teams Admin (Teams data review)
