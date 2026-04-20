# Teams Admin — Guest & External Access

## Guest Access (B2B Collaboration)

### What Guests Can Do (When Enabled)
- Join teams and channels as full team members
- Participate in chat conversations and meetings
- Access shared files in SharePoint and OneDrive
- Use apps and bots configured for the team
- Be added to individual teams by team owners

### What Guests Cannot Do (By Default)
- Create teams or discover teams via search
- Add apps, bots, or connectors to teams
- See org chart or full calendar of internal users
- Access resources outside their assigned teams

### Guest Access Controls
- Org-wide toggle: Teams Admin Center → Org-wide settings → Guest access (on/off)
- Per-team: team owners can allow or block guests for their specific team
- Guest capabilities: configure calling, meeting, messaging permissions separately
- Entra ID: External collaboration settings control which domains can be invited
- Conditional access: policies scoped to "Guest and external users" workload

### Guest Lifecycle Management
- Guest accounts created in Entra ID when first invited
- Access reviews: periodic review of guest access via Entra ID Access Reviews
- Guest expiration: can set B2B collaboration guest user expiration (30-365 days)
- Inactive guest cleanup: identify and remove guests with no recent sign-in activity
- Sponsor assignment: designate internal users responsible for guest access decisions

## External Access (Federation)

### Configuration
- Allows chat and presence with users from other Teams or Skype for Business organizations
- No team membership — limited to direct 1:1 and group chat, calls, and meetings
- Three modes: Open federation (all domains), Allow specific domains only, Block specific domains
- Managed in Teams Admin Center → External access

### Capabilities and Limitations
- External users can: chat, call, share screen, see presence status
- External users cannot: access team channels, files, apps, or team resources
- Compliance: host org retention and DLP policies apply to external chat messages
- Each organization independently controls whether to allow external access

## B2B Direct Connect (Shared Channels)

### How It Works
- Share a channel with external users or entire external teams without guest accounts
- External users authenticated by their home tenant, authorized by host tenant
- Appears in the external user's Teams client alongside their own teams

### Configuration
- Requires cross-tenant access settings in Entra ID on both sides
- Inbound: which external orgs' users can access your shared channels
- Outbound: which external orgs your users can access shared channels in
- Both orgs must explicitly enable B2B direct connect for each other

### Compliance in Shared Channels
- DLP policies from the host team's org apply to all participants including external
- Retention policies from host org apply to shared channel content
- Conditional access from host org applies (MFA, compliant device requirements)
- Sensitivity-label-encrypted files may not be accessible to external users unless permissions granted

## Cross-Tenant Access Settings (Entra ID)

### Configuration Location
- Entra ID → External Identities → Cross-tenant access settings

### Settings Per External Org
- Inbound access: control which external users can access your resources (B2B collab, B2B direct connect)
- Outbound access: control which of your users can access external org resources
- Trust settings: trust MFA, compliant device, hybrid Azure AD join claims from external org
- Default settings apply to all orgs not explicitly configured
