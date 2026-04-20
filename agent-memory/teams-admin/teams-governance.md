# Teams Admin — Teams Governance

## Team Creation Policies

### Who Can Create Teams
- By default: all users with an M365 license can create teams
- Restrict creation: create an Entra ID security group; only members can create M365 groups (and therefore Teams)
- Configuration: Entra ID → Groups → General → Group creation settings
- Restriction applies to all M365 group creation (Teams, Outlook groups, SharePoint sites, Planner)
- Admins (Global Admin, Teams Admin) can always create teams regardless of restriction

### Provisioning Alternatives
- Self-service with governance: allow creation but enforce naming, classification, expiration
- Request-based: users submit request, admin or automation provisions the team
- Template-based: pre-defined team structures with channels, apps, tabs

## Naming Conventions

### Entra ID Group Naming Policy
- Prefix/suffix rules applied automatically to all new M365 group display names
- Prefix types: fixed string ("Team-"), attribute-based ("{Department}-")
- Suffix types: fixed string ("-Project"), attribute-based ("-{CountryOrRegion}")
- Example: "{Department}-{GroupName}-Project" → "Marketing-Campaign2024-Project"

### Blocked Words
- Custom blocked word list prevents specific words in group names
- Use for: reserved terms, inappropriate words, brand names, department abbreviations
- Up to 5,000 blocked words supported
- Checked against display name and mail alias

## Classification and Sensitivity Labels

### Sensitivity Labels for Teams
- Applied at team creation to control: privacy (public/private), guest access, external sharing
- Can also control: unmanaged device access, conditional access settings
- Created in Microsoft Purview → Information protection → Labels
- Published to users via label policies

### Legacy Classification
- Text-only labels with no enforcement (predefined strings like "Internal", "Confidential")
- No longer recommended — migrate to sensitivity labels for actual enforcement
- Classification visible in Teams but does not restrict behavior

## Expiration Policies

### Configuration
- Entra ID → Groups → Expiration settings
- Lifetime options: 30, 60, 90, 180, 365 days, or custom value
- Applies to: all M365 groups, selected groups, or no groups

### Behavior
- Owners notified at: 30 days, 15 days, and 1 day before expiration
- If no owner action: group (and team) soft-deleted → 30-day recovery window
- Active teams auto-renew based on: user activity in Teams, SharePoint, Outlook, Planner
- Ownerless groups: configure fallback notification to secondary contacts or admin

## Guest Access Review

### Periodic Review
- Entra ID Access Reviews: schedule recurring reviews of guest access to teams
- Reviewers: team owners, designated reviewers, or self-review by guests
- Actions on denied review: remove guest from team, disable account, or block sign-in
- Best practice: quarterly guest access review with team owner as reviewer

## Channel Management

### Channel Types
| Type | Scope | Storage | Membership |
|------|-------|---------|------------|
| **Standard** | Visible to all team members | Team SharePoint site | All team members |
| **Private** | Visible to selected members only | Separate SharePoint site collection | Explicitly added members |
| **Shared** | Shared across teams and tenants | Host team's SharePoint site | Cross-team, cross-tenant members |

### Governance Considerations
- Private channels: separate compliance boundary (own SharePoint site, own retention scope)
- Shared channels: host org's DLP and retention policies apply to all participants
- Channel creation: can restrict who creates private/shared channels via meeting policies
- Maximum channels per team: 200 standard, 30 private, 30 shared

## Archive and Delete Policies

### Archival
- Read-only state: all content preserved, no new messages or file changes
- Archive via: Teams Admin Center, PowerShell (`Set-TeamArchivedState`), or Graph API
- Archived teams still consume licenses and SharePoint storage
- SharePoint site can optionally be set to read-only alongside archive

### Deletion
- Soft-delete: team removed from view, 30-day recovery window via Entra ID
- Hard-delete: permanent after 30 days (or admin purge via PowerShell)
- Associated resources follow: SharePoint site, Exchange mailbox, Planner plans, OneNote notebook
- Deletion does not immediately free storage — retention policies may preserve content

## Template Management

### Team Templates
- Pre-defined team structures: channels, apps, tabs, settings
- Built-in templates: project management, event management, retail, healthcare
- Custom templates: create from existing team or from scratch in Teams Admin Center
- Template policies: control which templates are visible to which users
- Templates enforce initial structure but do not prevent changes after creation