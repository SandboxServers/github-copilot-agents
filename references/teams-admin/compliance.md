# Teams Admin — Compliance

## Compliance in Teams

### Retention Policies
- Applied via Microsoft Purview → Teams chat messages and/or channel messages
- **Backend storage**: Chat → user's hidden Exchange mailbox folder; Channel → group mailbox; Shared channel → SubstrateGroup mailbox
- Actions: Retain only, Delete only, Retain then delete
- Scope: All users (org-wide) or specific users/groups
- Teams retention ≠ Exchange retention — separate policy types
- **Gotcha**: Up to 24-hour delay before Teams content is discoverable for compliance
- **Gotcha**: Deleting a message from Teams UI doesn't delete it from compliance storage if retention applies

### Data Loss Prevention (DLP)
- Prevents sharing sensitive info (SSN, credit card, custom patterns) in chats and channels
- Messages containing sensitive data: blocked, or sent with policy tip warning
- DLP in shared channels: **host team's policy applies** to all participants (including external)
- Requires Microsoft Purview DLP license (included in E5, available as add-on)

### eDiscovery & Legal Hold
- Content search across Teams messages, files, meeting recordings
- Legal hold: preserves all content even if user deletes it
- In-Place Hold (targeted) or Litigation Hold (everything)
- eDiscovery can search: chat messages, channel messages, meeting/call summaries, files
- **Gotcha**: Teams messages in eDiscovery may take up to 24 hours to appear

### Communication Compliance
- Scan for: offensive language, sensitive information, regulatory violations
- Covers: public/private channel messages, individual chats, attachments
- Policies created in Microsoft Purview, apply to Teams content
- Reviewers can investigate, remediate, escalate flagged content

### Information Barriers
- Prevent specific users/groups from communicating with each other
- Common in: financial services (Chinese wall), education, legal
- Affects: 1:1 chats, group chats, team-level membership
- Configured in Microsoft Purview, enforced in Teams

### Sensitivity Labels
- Applied to teams (the container) to control: privacy (public/private), guest access, external sharing, unmanaged device access
- Channel-level document sensitivity labels from host org apply in shared channels
- Files encrypted by sensitivity labels may not be openable by external users unless permissions granted
