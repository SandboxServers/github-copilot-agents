# Teams Admin — Admin Centers & Policies

## Admin Centers & How They Interact

| Admin Center | What It Controls for Teams | URL |
|--------------|---------------------------|-----|
| **Teams Admin Center** | Policies (meeting, messaging, calling, app), phone system, devices, analytics, user management | admin.teams.microsoft.com |
| **Entra ID (Azure AD)** | User identity, conditional access, guest access, app registrations, service principals | entra.microsoft.com |
| **Intune (Endpoint Manager)** | Device compliance, app protection policies, Teams device management (phones, displays, rooms) | intune.microsoft.com |
| **Microsoft Purview** | Retention, DLP, eDiscovery, communication compliance, sensitivity labels, information barriers | purview.microsoft.com |
| **Exchange Admin Center** | Shared mailboxes backing Teams (group mailboxes), mail flow affecting Teams notifications | admin.exchange.microsoft.com |
| **SharePoint Admin Center** | File storage backing Teams channels, sharing policies, site settings | admin.sharepoint.com |

### Key Interaction Points
- Teams **messages** are stored in Exchange mailboxes (UserMailbox for chats, GroupMailbox for channels, SubstrateGroup for shared channels)
- Teams **files** live in SharePoint (channel files) and OneDrive (chat files)
- **Conditional access** policies in Entra ID gate Teams access (MFA, compliant device, location)
- **Purview policies** (retention, DLP) apply across Exchange/SharePoint/Teams as a unified fabric
- **Intune compliance** feeds into conditional access → non-compliant device → blocked from Teams

## Policy Framework

### Policy Types

| Policy Type | Scope | Key Settings |
|-------------|-------|-------------|
| **Meeting policies** | Per-organizer, per-user, or both | Who can bypass lobby, recording, transcription, screen sharing, AI features, anonymous join |
| **Messaging policies** | Per-user | Edit/delete sent messages, chat permissions, Giphy/memes/stickers, read receipts, priority notifications |
| **Calling policies** | Per-user | Private calling, call forwarding, simultaneous ring, voicemail routing, delegation, call park, busy-on-busy |
| **App policies** | Per-user | Which apps allowed/blocked, pinned apps on app bar, app installation on behalf of users |
| **Event policies** | Per-organizer | Webinar and town hall settings, registration, Q&A, attendee limits |
| **Live events policies** | Per-user (legacy) | Who can schedule, transcription, recording, attendee engagement |

### Policy Inheritance & Assignment Model
```
Priority (highest wins):
1. Direct user assignment (explicit policy assigned to individual user)
2. Group assignment (policy assigned to security group / M365 group)
   └─ If user is in multiple groups → lowest group rank number wins
3. Global (Org-wide default) policy

Key rules:
- Changes take up to 24 hours to propagate (sometimes faster, never instant)
- You can't delete a policy while users are assigned to it
- Policy packages = bundles of policies for common roles (e.g., FirstlineWorker, Educator)
- Batch assignment via PowerShell for scale (New-CsBatchPolicyAssignmentOperation)
```

### Unified Settings & Policies (New Experience — Spring 2024+)
- Single "Settings & policies" tab in Teams Admin Center
- Two sub-tabs: **Org-wide default settings** and **Custom policies for users & groups**
- Three-tab structure per custom policy: Edit Policy → Assigned Groups → Assigned Users
- Replaces the scattered per-feature navigation

## Key PowerShell Modules

| Module | Use |
|--------|-----|
| `MicrosoftTeams` | Teams-specific management (policies, phone, users) |
| `Microsoft.Graph` | Broader M365 management including Teams (groups, users, compliance) |
| `ExchangeOnlineManagement` | Mailbox/transport rules affecting Teams |

### Common Administrative Cmdlets
```powershell
# Policy management
Get-CsTeamsMeetingPolicy
New-CsTeamsMeetingPolicy -Identity "RestrictedMeetings" -AllowMeetingRecording $false
Grant-CsTeamsMeetingPolicy -Identity user@contoso.com -PolicyName "RestrictedMeetings"

# Batch policy assignment
New-CsBatchPolicyAssignmentOperation -PolicyType TeamsMeetingPolicy -PolicyName "RestrictedMeetings" -Identity @("user1@contoso.com","user2@contoso.com")

# Phone system
New-CsOnlineVoiceRoute -Identity "SeattleRoute" -NumberPattern "^\+1(425|206)\d{7}$" -OnlinePstnGatewayList "sbc1.contoso.com"
Set-CsPhoneNumberAssignment -Identity user@contoso.com -PhoneNumber "+14255551234" -PhoneNumberType DirectRouting

# Team lifecycle
Get-Team -DisplayName "Project Alpha"
Set-TeamArchivedState -GroupId <id> -Archived $true
```

## Common Mistakes & Anti-Patterns

1. **Not planning policy inheritance** → users get unexpected combinations of global + group + direct policies
2. **Enabling guest access without Entra ID external collaboration controls** → anyone can be invited from any domain
3. **No naming convention enforcement** → 500 teams named "Marketing" with no way to distinguish them
4. **Skipping CQD building data upload** → can't correlate call quality issues to physical locations
5. **Direct Routing without redundant SBCs** → single SBC failure = no PSTN calling for entire org
6. **No team expiration policy** → thousands of abandoned teams consuming storage and creating confusion
7. **Applying DLP policies without testing** → legitimate business communications blocked, users lose trust
8. **Ignoring shared channel compliance implications** → host team DLP policy applies to external users (may be too restrictive or too permissive)
9. **Resource accounts without phone numbers** → Auto Attendants/Call Queues unreachable from PSTN
10. **Not restricting team creation** → team sprawl, duplicate teams, ungoverned data
