# Teams Admin — Meeting Policies

## Meeting Policies

### Policy Scope
- Applied per-organizer (controls what the meeting allows) or per-user (controls what the user can do in any meeting)
- Assigned via: Teams Admin Center, PowerShell (`Grant-CsTeamsMeetingPolicy`), or group assignment

### Key Policy Settings
| Setting | Options | Default |
|---------|---------|---------|
| **Who can present** | Everyone, people in org, specific people, only organizer | Everyone |
| **Recording** | Allow/block cloud recording | Allowed |
| **Transcription** | Allow/block live transcription | Allowed |
| **Lobby bypass** | Everyone, people in org, people in org and trusted, invited users only | People in my org |
| **Meeting chat** | Enabled, disabled, in-meeting only | Enabled |
| **Reactions** | Allow/block reactions and hand raise | Allowed |
| **Screen sharing** | Entire screen, single app, disabled | Entire screen |
| **Anonymous join** | Allow/block users without accounts | Allowed |
| **IP audio/video** | Allow/block audio and video in meetings | Allowed |
| **Copilot** | Allow/block Copilot in meetings | Varies by license |

## Meeting Settings (Org-Wide)

### Network and Media
- Custom meeting backgrounds: upload org-approved background images
- NDI (Network Device Interface): enable for broadcast-quality production
- QoS markers: enable DSCP marking for meeting media traffic
- Media port ranges: configure for QoS (audio, video, screen sharing)

### Email and Calendar
- Meeting join link format and customization
- Calendar integration settings
- Allow Outlook add-in for Teams meetings

## Meeting Templates

### Purpose
- Pre-configure meeting options so organizers don't have to set them manually
- Templates enforce or default specific settings (e.g., lobby bypass, recording, chat)

### Configuration
- Created in Teams Admin Center → Meetings → Meeting templates
- Settings can be: locked (organizer cannot change) or default (organizer can override)
- Assign templates to specific user groups or make available org-wide
- Combine with sensitivity labels for security-focused meeting types

## Sensitivity Labels for Meetings

### What They Control
- Who can bypass the lobby
- Who can present and share screen
- Whether recording and transcription are allowed
- End-to-end encryption enforcement
- Meeting chat permissions
- Watermarking on video and shared content
- Copilot usage in the meeting

### Requirements
- Microsoft 365 E5, E5 Compliance, or E5 Information Protection license
- Labels created in Microsoft Purview → Information protection
- Published to users via label policies

## Webinars

### Configuration
- Webinar policies control: who can schedule webinars, registration, attendee limits
- Registration: required or optional; captures attendee info before the event
- Presenter and organizer controls: manage backstage, Q&A, attendee engagement
- Attendee experience: view-only with Q&A and reactions (no mic/camera by default)

### Capacity
- Standard webinars: up to 1,000 attendees
- Premium webinars (Teams Premium): up to 1,000 with advanced features

## Town Halls

### Configuration
- Large-scale broadcast events (replaces Live Events)
- Capacity: up to 10,000 attendees (up to 20,000 with Teams Premium)
- Attendee experience: watch-only with Q&A
- Producer controls: manage presenters, layouts, live event flow

### Settings
- Town hall policies: who can schedule, Q&A, recording, captions
- RTMP-in: external encoder support for professional broadcast production
- eCDN: enterprise content delivery network for optimizing bandwidth in large orgs

## Teams Premium Meeting Features

### Watermarking
- Overlay attendee email on video feeds and shared content
- Prevents unauthorized screen capture/photography
- Configured via meeting template or sensitivity label

### Custom Branding
- Custom meeting backgrounds, logos, and themes
- Brand the pre-join and in-meeting experience with org identity

### Advanced Protection
- End-to-end encryption for meetings (limits some features like recording, transcription)
- Who can record: restrict to organizers only
- Prevent copy/paste in meeting chat
- Sensitivity label enforcement for meeting content