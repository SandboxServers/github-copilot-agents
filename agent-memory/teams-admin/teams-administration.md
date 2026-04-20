# Teams Admin — Teams Administration

## Teams Admin Center

### Overview
- Primary management portal for Microsoft Teams: admin.teams.microsoft.com
- Manages: policies, users, devices, phone system, analytics, org-wide settings
- Integrates with: Entra ID (identity), Purview (compliance), Intune (devices), Exchange (mailboxes), SharePoint (files)

### Key Sections
- Users: manage individual user settings, policy assignments, call history
- Teams & Groups: view/manage teams, membership, settings
- Meetings: meeting policies, settings, bridge configuration
- Messaging: messaging policies, chat settings
- Voice: phone system, call routing, dial plans, emergency policies
- Devices: Teams Rooms, phones, displays, panels
- Analytics & Reports: usage, call quality, device health

## Policy Types

### Policy Categories
| Policy Type | Controls | Assigned To |
|-------------|----------|-------------|
| **Meeting policies** | Recording, transcription, lobby, screen sharing, who can present | Per-organizer or per-user |
| **Messaging policies** | Edit/delete messages, chat, Giphy, memes, read receipts, priority notifications | Per-user |
| **Calling policies** | Private calls, forwarding, delegation, simultaneous ring, voicemail, busy-on-busy | Per-user |
| **App policies** | Which apps allowed/blocked, pinned apps in app bar | Per-user |
| **Live events policies** | Who can schedule, transcription, recording for live events | Per-user |
| **Event policies** | Webinar and town hall configuration, registration, Q&A | Per-organizer |

## Policy Assignment Model

### Priority Order (Most Specific Wins)
```
1. Direct user assignment — policy explicitly assigned to individual user
2. Group assignment — policy assigned to security group or M365 group
   └─ If user is in multiple groups: lowest rank number wins
3. Global (Org-wide default) — applies to all users without explicit assignment
```

### Assignment Methods
- Teams Admin Center: assign to individual users or groups via UI
- PowerShell: `Grant-CsTeamsMeetingPolicy -Identity user@contoso.com -PolicyName "PolicyName"`
- Batch assignment: `New-CsBatchPolicyAssignmentOperation` for bulk operations
- Policy packages: pre-built bundles for roles (Frontline Worker, Educator, Clinical Worker)

### Propagation
- Policy changes take minutes to hours to propagate (up to 24 hours in worst case)
- Cannot delete a policy while users are still assigned to it

## Teams Settings

### Guest Access
- Org-wide toggle: Teams Admin Center → Org-wide settings → Guest access
- Controls: whether external users can be added as team members
- Granular settings: guest calling, meeting, messaging capabilities

### External Access (Federation)
- Controls: chat and presence with users from other Teams/Skype for Business orgs
- Modes: allow all domains, allow specific domains, block specific domains

### Email Integration
- Channel email addresses: users can send email to a channel
- Configurable: who can send email to channels (any sender vs domain-restricted)

## Teams Lifecycle

### Lifecycle Stages
```
Creation → Active Management → Archival → Deletion

Creation:
- Manual (users/admins), templates, provisioning tools, Graph API
- Apply naming conventions, sensitivity labels, classification at creation

Active Management:
- Membership changes, channel management, app configuration
- Monitor activity via usage reports

Archival:
- Read-only state — all content preserved, no new posts or file changes
- Set-TeamArchivedState -GroupId <id> -Archived $true
- Can be unarchived if needed

Deletion:
- Soft-delete: 30-day recovery window
- Hard-delete: permanent after 30 days (or admin purge)
- Associated SharePoint site and mailbox follow same lifecycle
```

### Naming Conventions
- Enforced via Entra ID group naming policy
- Prefix/suffix: department, location, project type (e.g., "Team-{Department}-{GroupName}")
- Blocked words: prevent inappropriate or reserved names
- Applies to all M365 groups (Teams, Outlook groups, SharePoint sites)

### Expiration Policies
- Configured in Entra ID → Groups → Expiration
- Options: 30, 60, 90, 180, 365 days, or custom value
- Owners notified before expiration (30 days, 15 days, 1 day)
- Expired teams soft-deleted with 30-day recovery window
- Activity-based renewal: teams with active usage auto-renew