# Teams Administration Best Practices

Proven practices for governing, securing, and operating Microsoft Teams at enterprise scale.

---

## Governance: Team Lifecycle Management

- **Restrict team creation** to a designated security group — prevent unchecked sprawl by funneling requests through an approval process or self-service portal
- **Naming convention:** Enforce M365 group naming policy with prefixes/suffixes (e.g., `PROJ-{GroupName}-{Department}`) to standardize team names across the tenant
- **Expiration policy:** Set 90 or 180-day group expiration — owners receive renewal prompts; unrenewed teams are soft-deleted (30-day recovery window)
- **Archive inactive teams:** Teams unused for 90+ days should be archived (read-only) rather than deleted — preserves content for compliance while decluttering the active team list
- **Ownerless group policy:** Configure automatic notifications to members when all owners leave; escalate to IT if no new owner is assigned within a defined period
- **Guest access reviews:** Use Entra ID access reviews to periodically revalidate guest membership in teams — remove guests who no longer need access

## Security: Information Protection and Access Controls

- **Sensitivity labels:** Apply container-level labels (Public, Internal, Confidential, Highly Confidential) that control guest access, sharing settings, and conditional access per classification
- **DLP policies:** Extend Microsoft Purview DLP to Teams chat and channel messages — detect and block sharing of credit card numbers, SSNs, health records, and custom sensitive info types
- **Conditional access for Teams:** Create CA policies targeting the Teams app — require MFA, compliant device, and trusted network for access to Teams on unmanaged devices
- **Information barriers:** For regulated industries (financial services, legal), configure information barriers to prevent communication between conflicting groups
- **Safe links and safe attachments:** Ensure Microsoft Defender for Office 365 policies cover Teams — files shared in Teams are scanned; malicious links are blocked at click time
- **External sharing controls:** Configure SharePoint and OneDrive sharing policies in tandem with Teams — Teams file sharing inherits SharePoint sharing settings, not Teams-specific controls
- **Session controls:** Use Conditional Access app control (via Defender for Cloud Apps) to enforce download restrictions, prevent copy/paste, and apply session-based monitoring for sensitive teams

## Voice: Teams Phone Design

- **Dial plan design first:** Document the full PSTN dial plan before configuring anything — number ranges, emergency numbers, extension dialing, normalization rules
- **Auto attendants and call queues:** Design multi-level call flows with business-hours/after-hours routing, holiday schedules, overflow actions, and timeout-to-voicemail
- **Redundant SBCs:** For Direct Routing, deploy SBC pairs in active/active or active/passive with failover — test failover quarterly
- **Operator Connect or Teams Calling Plans** for simpler deployments — evaluate before defaulting to Direct Routing; lower operational burden for orgs without existing SBC infrastructure
- **Emergency calling:** Register validated civic addresses for all office locations; configure emergency calling policies with security desk notification; enable dynamic emergency calling for remote workers
- **Number management:** Maintain a documented inventory of all phone numbers (service numbers, user numbers, toll-free) with assignment status — track in a CMDB or spreadsheet to avoid number conflicts
- **Call analytics baseline:** Establish baseline call quality metrics per location during initial deployment — use these baselines to detect regression after network or configuration changes
- **Common area phones:** Use dedicated resource accounts with Teams Shared Devices or Common Area Phone licenses — do not waste full user licenses on lobby phones

## Quality: Monitoring and Network Readiness

- **CQD building data:** Upload building, subnet, and network data to Call Quality Dashboard — this enables location-based quality correlation and is the single most impactful CQD setup step
- **QoS marking:** Configure DSCP marking on endpoints and network infrastructure — prioritize audio (EF/46), video (AF41/34), and screen sharing (AF21/18) traffic
- **Network assessment:** Run the Teams Network Assessment Tool before deployment and periodically — validates bandwidth, latency, jitter, and packet loss against Microsoft baselines
- **Monthly CQD review:** Establish a cadence for reviewing poor call rate, audio poor stream percentage, and setup failure rates — set thresholds and investigate when exceeded
- **Real-time telemetry:** Use Real-Time Analytics in Teams admin center for live meeting monitoring during critical events
- **Bandwidth planning:** Ensure 1.5 Mbps per concurrent video call (minimum); plan for peak concurrent usage with network capacity modeling

## Compliance: Retention, eDiscovery, and Communication Compliance

- **Retention policies:** Create Purview retention policies targeting Teams channel messages and Teams chats separately — regulatory requirements often mandate 3-7 year retention for financial services
- **eDiscovery readiness:** Ensure Teams data (chats, channel messages, meeting recordings, transcripts) is included in eDiscovery content search locations; test regularly with mock legal holds
- **Communication compliance:** Enable Purview Communication Compliance for regulated industries — detect policy violations (insider trading language, harassment) in Teams messages
- **Audit logging:** Verify Teams events are captured in the Microsoft 365 unified audit log — admin actions, user activities, and app events should all be searchable
- **Litigation hold:** Understand that Teams chat data subject to litigation hold is preserved in the user's Exchange Online mailbox (substrate holds); channel messages are held in the group mailbox

## App Governance: Permission Policies and Org-Wide Settings

- **App permission policies:** Create tiered policies — IT staff get all apps; general users get Microsoft + vetted third-party; frontline gets a curated minimal set
- **Org-wide app settings:** Disable "Allow interaction with custom apps" for users who should not sideload — keep it enabled only for developers/IT
- **App setup policies:** Pin essential apps (Shifts, Approvals, Tasks) to specific user groups; remove unused default pins to simplify the experience
- **Review third-party apps:** Check publisher attestation, Microsoft 365 certification, and requested Graph permissions before allowing any third-party app
- **Block by default:** For highly regulated environments, set org-wide to "Block all third-party apps" and explicitly allow vetted apps — safer than allowing all and trying to block bad ones
- **Custom app governance:** If developers build custom Teams apps (LOB apps), require a review and approval process before publishing to the org app catalog
- **App usage reporting:** Monitor app usage analytics in Teams admin center — identify unused allowed apps and widely used blocked apps to refine policies

## Rooms and Devices: Standardized Management

- **Standardized hardware:** Select from Microsoft-certified Teams Rooms hardware (Poly, Yealink, Logitech, Neat) — maintain a supported hardware catalog to simplify procurement and support
- **Teams Rooms Pro:** Use Teams Rooms Pro licensing for centralized device management, health monitoring, and remote troubleshooting via the Pro Management Portal
- **Firmware management:** Enroll devices in Intune for centralized firmware and OS updates; test updates on pilot rooms before broad rollout
- **Monitoring dashboards:** Use the Teams Rooms Pro Management Portal or Teams admin center device health page to monitor room availability, call quality, and hardware status
- **Resource account setup:** Create dedicated resource accounts for each room with proper Teams Rooms licensing; configure calendar processing rules for automatic meeting acceptance
- **Room standards:** Define and document room configurations (camera placement, microphone coverage, display sizing) by room type (huddle, medium, large, boardroom) for consistent user experience
- **Physical security:** Ensure room devices are physically secured — USB ports should be restricted to prevent unauthorized peripheral connections; configure device lock policies

## Change Management and Updates

- **Update policies:** Assign Teams update policies to control the update ring — use Preview for IT pilot, Current Channel for general population, Monthly Enterprise Channel for change-sensitive environments
- **Message Center monitoring:** Subscribe to Microsoft 365 Message Center notifications for Teams — track upcoming feature changes, deprecations, and policy defaults with at least 30-day lead time
- **User communication:** Establish a regular cadence (monthly or per-release) for communicating Teams changes to end users — include what changed, why, and any action needed
- **Training and adoption:** Provide role-specific Teams training — executives, meeting organizers, call queue agents, and frontline workers have different needs; leverage Microsoft's adoption resources and Champions program
- **Feature management:** Use Teams policies to control feature rollout — disable features that conflict with compliance requirements before they reach end users
