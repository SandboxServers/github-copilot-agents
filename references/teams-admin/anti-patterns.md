# Teams Administration Anti-Patterns

Common Microsoft Teams administration misconfigurations that lead to sprawl, security gaps, and poor user experience.

---

## No Governance — Anyone Can Create Teams

- Default M365 settings allow every user to create Microsoft 365 Groups (and therefore Teams)
- Result: hundreds of duplicate, abandoned, and poorly named teams within months
- No naming convention means `Marketing`, `Marketing Team`, `Mktg`, `Marketing-2024` all coexist
- No expiration policy means inactive teams persist indefinitely, consuming storage and cluttering search
- **Fix:** Restrict team creation to a security group via `Set-AzureADDirectorySetting`; enforce naming policies and 90/180-day expiration with owner renewal prompts

## Default Policies for Everyone

- Using only the Global (Org-wide default) policy for meetings, messaging, calling, and apps
- All users — executives, frontline workers, call center agents — get identical capabilities
- No role-based policy packages assigned (Microsoft provides packages for frontline, education, healthcare)
- **Fix:** Create targeted policies for each persona; assign via group-based policy assignment in Teams admin center or `Grant-CsTeamsMeetingPolicy`

## Allowing All Third-Party Apps Without Review

- Org-wide app settings left at "Allow all apps" — no curation or review process
- Users install unvetted apps that can access Teams data (messages, files, calendars)
- No app permission policies scoping which apps are available to which user groups
- **Fix:** Set org-wide to "Allow specific apps" or "Block specific apps"; use app permission policies per group; review publisher attestation and M365 certification status

## Not Configuring Meeting Policies

- Default meeting policies leave recording available to all organizers without retention awareness
- Transcription and live captions enabled or disabled globally without considering compliance needs
- No lobby controls — external participants join meetings directly without waiting
- Default "who can present" set to Everyone instead of role-appropriate defaults
- **Fix:** Create meeting policies per role; configure lobby bypass, recording defaults, transcription, and content sharing per policy

## External Access Wide Open Without Domain Restrictions

- External access (federation) left at "Open to all external domains" — any Teams/Skype for Business user can chat
- No allowed/blocked domain list — potential phishing vector via external chat
- Confused with guest access — external access provides chat/calling but no team membership
- **Fix:** Restrict external access to specific partner domains; use `Set-CsTenantFederationConfiguration -AllowedDomains` with explicit domain list

## No Compliance Policies

- No retention policies applied to Teams chats or channel messages
- Teams data not included in DLP policies — sensitive data shared in chat goes undetected
- eDiscovery not configured to search Teams content — legal hold gaps
- Communication compliance not enabled for regulated industries
- **Fix:** Create Purview retention policies targeting Teams; extend DLP policies to Teams chat and channels; include Teams in eDiscovery content locations

## Teams Proliferation and Stale Content

- Hundreds of teams created for one-off projects, events, or brainstorming sessions — never archived
- Stale teams appear in search results, confusing users and diluting content discovery
- Owners have left the organization — ownerless teams cannot be managed
- **Fix:** Deploy expiration policies; use access reviews for team membership; set up ownerless group policy to assign new owners; archive completed project teams

## Not Using Sensitivity Labels for Team Classification

- No sensitivity labels applied — all teams have identical access controls regardless of content sensitivity
- Confidential and public teams share the same guest access, sharing, and device access settings
- **Fix:** Create sensitivity labels (Public, Internal, Confidential, Highly Confidential) with container-level settings controlling guest access, sharing restrictions, and conditional access requirements

## Phone System Without Proper Call Routing Design

- Auto attendants with flat menus — 10+ DTMF options with no time-of-day routing
- Call queues with no overflow or timeout configuration — callers wait indefinitely
- No shared voicemail for after-hours calls — calls ring out with no capture
- Direct Routing deployed without redundant SBC pairs — single point of failure
- **Fix:** Design call flows with business-hours/after-hours routing, overflow to shared voicemail, and SBC HA pairs with failover; document full dial plan before implementation

## Ignoring CQD Data for Call Quality Issues

- Call Quality Dashboard (CQD) not configured or never reviewed after initial deployment
- Building and subnet data not uploaded — cannot correlate quality issues to physical locations
- Users report "Teams calls are bad" but no data-driven investigation is performed
- **Fix:** Upload building data to CQD; review CQD monthly for poor call rate trends; set up quality alerts; map subnets to buildings for location-based troubleshooting

## No Update Management Strategy

- Relying entirely on Microsoft's auto-update rings with no organizational awareness
- New features surprise users — UI changes with no communication or training
- Preview features enabled in production without testing
- **Fix:** Use Teams update policies to control update ring (Current, Monthly Enterprise); subscribe to Message Center; communicate changes proactively; use preview ring for IT pilot group only

## Emergency Calling Without Validated Addresses

- E911 emergency calling deployed without validated civic addresses for all locations
- Dynamic emergency calling not configured — remote workers dial 911 with no location context
- No notification to security desk when emergency calls are placed
- Emergency call routing policies not assigned to users with calling plans or Direct Routing
- **Fix:** Register and validate all civic addresses in Teams admin center; configure emergency calling policies with notification groups; enable dynamic emergency calling for remote/mobile users; test quarterly
