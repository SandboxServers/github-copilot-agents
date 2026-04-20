# Teams Administration Gotchas

Non-obvious behaviors and subtle distinctions that frequently trip up Teams administrators.

---

## Policy Assignment Hierarchy

- User-level policy overrides group-level policy, which overrides org-wide (Global) default
- Precedence order: **Direct user assignment > Group assignment (by group assignment rank) > Global policy**
- Group assignment rank matters when a user is in multiple groups — lower rank number wins
- Policy changes can take **up to 24 hours** to propagate to all users
- Use `Get-CsUserPolicyAssignment -Identity user@contoso.com` to see effective policy and assignment source

## Teams Premium Licensing Requirements

- Some meeting features (custom branded meetings, watermarks, end-to-end encryption for meetings, RTMP-in) require Teams Premium
- Custom meeting templates and sensitivity label enforcement on meetings require Teams Premium
- Advanced webinar capabilities (registration waitlists, presenter bios) are Teams Premium only
- AI-powered features like intelligent meeting recap require Copilot for Microsoft 365 or Teams Premium
- Check the feature matrix — assuming a feature is in standard licensing causes deployment surprises

## Direct Routing SBC Certificate Requirements

- SBC certificates must be from a **publicly trusted CA** — self-signed or internal PKI certificates are rejected
- The SAN (Subject Alternative Name) must match the SBC FQDN exactly as configured in Teams
- Wildcard certificates are supported but only if the SBC supports them and the wildcard matches the FQDN pattern
- Certificate chain must be complete — intermediate CA certificates must be included in the SBC configuration
- Certificate renewal requires careful timing — expired SBC cert means all PSTN calls fail immediately

## Meeting Recordings Storage Location

- Meeting recordings are stored in **OneDrive for Business** (non-channel meetings) or **SharePoint** (channel meetings)
- Recordings are NOT stored in Teams itself — Teams just provides a link to the file in OneDrive/SharePoint
- Recording ownership follows the meeting organizer's OneDrive; if the organizer leaves, recordings become orphaned
- Retention policies for Teams chat/channels do **not** cover recording files — you need OneDrive/SharePoint retention policies
- Recording expiration (auto-delete) is configurable via meeting policy: `Set-CsTeamsMeetingPolicy -MeetingRecordingExpirationDays`

## Guest Access vs External Access

- **Guest access:** user is added as a guest member to a team — can access channels, files, apps, and chat within that team; appears in the team roster; requires Entra ID B2B invite
- **External access (federation):** user can chat and call from their own tenant — no team membership, no file access, no app access; appears as "[External]" in chat
- Disabling guest access does NOT disable external access — they are controlled independently
- External access is tenant-wide by domain; guest access policies are per-user and per-team
- Shared channels use Entra ID B2B direct connect — a third model distinct from both guest and external access

## Teams Rooms Licensing

- Teams Rooms devices require a **separate** Teams Rooms license — either Teams Rooms Basic (free, up to 25 rooms) or Teams Rooms Pro
- A standard Teams user license assigned to a room account does **not** unlock room-specific features (remote management, device health monitoring)
- Teams Rooms Pro includes Intune enrollment, remote device management, and advanced analytics
- Surface Hubs have their own licensing considerations — not covered by Teams Rooms licenses
- Phone devices (common area phones) need a **Teams Shared Devices** license or **Common Area Phone** license — not a Teams Rooms license

## Compliance Recording Requires Third-Party Solution

- Teams does not offer built-in compliance recording (full call/meeting recording for regulatory purposes)
- Compliance recording requires a **Microsoft-certified third-party solution** (e.g., Verint, NICE, ASC)
- The compliance recording policy routes media to the third-party recorder via bot-based injection
- Compliance recording is distinct from convenience recording (the "Record" button in meetings)
- Policy-based compliance recording cannot be declined by participants — it records automatically

## App Permission Policies Don't Apply Retroactively

- If you block an app in app permission policies, users who **already installed** the app retain access
- The policy only prevents **new installations** — existing installs must be removed separately
- Use `Remove-TeamsAppInstallation` PowerShell cmdlet to uninstall apps from users who already have them
- This behavior is by design but frequently surprises admins who expect blocking to immediately revoke access

## Voicemail Requires Exchange Online Mailbox

- Teams voicemail (Cloud Voicemail) requires the user to have an **Exchange Online mailbox**
- Users with on-premises Exchange mailboxes only get voicemail if Exchange hybrid is properly configured with OAuth
- Shared mailboxes and resource accounts for auto attendants/call queues may need mailbox configuration for shared voicemail
- Voicemail transcription is a separate setting in the calling policy — can be enabled or disabled independently
- Voicemail messages are stored in the user's Exchange mailbox, making them subject to Exchange retention policies, not Teams policies
