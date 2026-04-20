# Teams Admin — Compliance

## Retention Policies

### Configuration
- Created in Microsoft Purview compliance portal → Data lifecycle management
- Separate policy types for Teams chat messages and Teams channel messages
- Actions: Retain only, Delete only, Retain then delete (with configurable periods)
- Scope: Org-wide (all users) or targeted to specific users, groups, or sites

### Storage Locations
- Chat messages → user's hidden Exchange mailbox folder (TeamsMessagesData)
- Channel messages → group mailbox associated with the team
- Shared channel messages → SubstrateGroup mailbox
- Meeting recordings → OneDrive (1:1/group) or SharePoint (channel meetings)
- Teams retention policies are separate from Exchange retention — both can coexist

### Behavior
- Retained content preserved in SubstrateHolds folder even if user deletes from UI
- Deletion by policy removes from user view; compliance copy retained until hold expires
- Up to 24-hour delay before new content is discoverable for compliance workflows

## eDiscovery

### Content Search
- Search across: chat messages, channel messages, meeting/call summaries, files shared in Teams
- Uses Microsoft Purview eDiscovery (Standard or Premium)
- Premium adds: review sets, advanced analytics, conversation threading, custodian management

### Legal Hold
- Litigation Hold: preserves all mailbox content including Teams messages
- In-Place Hold (targeted): preserve content matching specific criteria
- Hold notification workflows available in eDiscovery Premium
- Teams messages may take up to 24 hours to appear in eDiscovery results

## Data Loss Prevention (DLP)

### Policy Configuration
- Created in Microsoft Purview → DLP policies → Teams chat and channel messages
- Detects sensitive information types: SSN, credit card numbers, custom regex patterns
- Actions: Block message, send with policy tip warning, notify compliance admin
- Supports sensitive information types, trainable classifiers, exact data match

### Scope and Behavior
- Applies to: 1:1 chats, group chats, standard channels, private channels, shared channels
- Shared channels: host team's DLP policy applies to all participants including external users
- Policy tips shown inline in Teams compose box before message is sent
- Requires Microsoft Purview DLP license (included in E5, available as add-on for E3)

## Communication Compliance

### Purpose
- Detect policy violations: offensive language, sensitive information, regulatory violations
- Covers: public/private channel messages, individual chats, attachments, meeting transcriptions

### Workflow
- Policies created in Microsoft Purview → Communication compliance
- Built-in classifiers: threat, harassment, discrimination, profanity
- Custom keyword dictionaries and sensitive info types supported
- Reviewers investigate flagged content → remediate, escalate, resolve, tag as false positive
- Integration with insider risk management for pattern detection

## Information Barriers

### Configuration
- Prevent specific user segments from communicating with each other in Teams
- Common scenarios: financial services (Chinese wall), education, legal, M&A
- Affects: 1:1 chats, group chats, team membership, meeting invitations, file sharing

### Implementation
- Define segments (based on user attributes like department, group membership)
- Create barrier policies between segments (block or allow)
- Configured in Microsoft Purview → Information barriers
- Enforced in Teams, SharePoint, and OneDrive
- Requires information barriers compliance processing to apply changes

## Audit Logging

### Teams Events Captured
- Team creation, deletion, membership changes, settings changes
- Channel operations (create, delete, update)
- Meeting events (scheduled, joined, policy changes)
- App installations and bot interactions
- Policy assignment changes, admin actions

### Access and Retention
- Viewed in Microsoft Purview → Audit (Standard or Premium)
- Standard audit: 180-day retention (E3/E5)
- Premium audit: up to 10-year retention policies, high-value event access
- Search by: activity type, user, date range, Teams-specific filters
- Export to CSV for analysis; integrate with SIEM via Management Activity API
