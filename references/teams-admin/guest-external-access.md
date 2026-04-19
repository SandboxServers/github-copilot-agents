# Teams Admin — Guest & External Access

## Guest Access & External Access

### Guest Access (B2B Collaboration)
```
What guests CAN do (when enabled):
- Join teams and channels
- Participate in chat and meetings
- Access shared files (SharePoint/OneDrive)
- Be added to individual teams by team owners

What guests CANNOT do (by default):
- Create teams
- Discover teams via search
- Add apps, bots, or connectors
- See org chart or calendar of internal users

Controls:
- Org-wide: Teams Admin Center → Org-wide settings → Guest access (on/off)
- Per-team: Team owners can allow/block guests per team
- Entra ID: External collaboration settings control which domains can be invited
- Conditional access: Policies scoped to "Guest and external users"
```

### External Access (Federation)
- Allows chat and meetings with users from other Teams/Skype for Business orgs
- No team membership — just direct communication
- Three modes: Open federation (all), Allow specific domains, Block specific domains
- Compliance: host org retention policies apply to external messages

### Shared Channels (B2B Direct Connect)
- Share a channel with external teams without guest accounts
- Users authenticated by their home org, access controlled by host org
- DLP/retention policies from host team apply
- Conditional access from host org applies (MFA, compliant device)
- External users can't open sensitivity-label-encrypted files unless permissioned
