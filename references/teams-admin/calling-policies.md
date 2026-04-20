# Teams Admin — Calling Policies & Phone System

## Teams Phone System

### Overview
- Cloud-based PBX replacing on-premises phone systems
- Requires Teams Phone license (included in E5, add-on for E3)
- Provides: call control, PBX capabilities, PSTN connectivity options

## PSTN Connectivity Options

### Microsoft Calling Plans
- Microsoft acts as PSTN carrier — fully cloud-managed
- Phone numbers provisioned directly from Microsoft
- Best for: simple deployments, no existing carrier contracts
- Available in limited countries; domestic and international plan options
- No on-premises infrastructure required

### Direct Routing
- Connect your own Session Border Controller (SBC) to Teams Phone
- SBC routes calls between Teams and PSTN via your carrier's SIP trunk
- Best for: complex environments, existing carrier contracts, regulatory requirements
- Requirements: certified SBC, TLS certificate, DNS configuration, firewall rules

### Direct Routing Architecture
```
PSTN ←→ Carrier SIP Trunk ←→ SBC ←→ Microsoft 365 (Teams Phone)
                                              ↕
                                        Teams Client

Network requirements:
- SIP signaling: TCP port 5061 (TLS) to/from SBC
- Media: UDP ports 49152-53247 between SBC and Microsoft media processors
- SBC FQDN must be publicly resolvable with valid TLS certificate
- Redundancy: deploy paired SBCs for high availability
```

### Operator Connect
- Carrier-managed PSTN connectivity through Microsoft's Operator Connect program
- Carrier manages SBC infrastructure — simpler than Direct Routing
- Best for: organizations wanting carrier-managed service without SBC management
- Configuration: select operator in Teams Admin Center, carrier provisions numbers

### Teams Phone Mobile
- Use existing mobile carrier phone number as Teams phone number
- SIM-enabled calling: native dialer and Teams share the same number
- Best for: mobile-first workforces
- Requires carrier participation in the Teams Phone Mobile program

## Voice Routing

### Dial Plans and Normalization Rules
- Dial plan: set of normalization rules translating user-dialed numbers to E.164 format
- Normalization rules: regex patterns matching dialed digits and transforming them
- Types: Tenant dial plan (admin-assigned), Service country dial plan (automatic)
- Common use: 4-digit extension dialing → translate to full +E.164 number
- Example: `^(\d{4})$` → `+1425555$1` (4-digit extension to full number)

### Voice Routing Policies
```
Voice Routing Policy (assigned to user)
  └─ PSTN Usage Records (containers linking policies to routes)
       └─ Voice Routes (number patterns + gateway priority)
            └─ Online PSTN Gateways (SBC endpoints)

Example flow:
User dials +1-425-555-1234
→ Matches voice routing policy assigned to user
→ PSTN usage "US-Local" selected
→ Voice route "SeattleRoute" matches pattern ^\+1(425|206)\d{7}$
→ Route sends to sbc1.contoso.com (priority 1), failover to sbc2.contoso.com
```

### Key PowerShell Commands
```powershell
New-CsOnlineVoiceRoute -Identity "SeattleRoute" -NumberPattern "^\+1(425|206)\d{7}$" `
  -OnlinePstnGatewayList "sbc1.contoso.com","sbc2.contoso.com"
New-CsOnlineVoiceRoutingPolicy -Identity "US-West" -OnlinePstnUsages "US-Local","US-LongDistance"
Grant-CsOnlineVoiceRoutingPolicy -Identity user@contoso.com -PolicyName "US-West"
Set-CsPhoneNumberAssignment -Identity user@contoso.com -PhoneNumber "+14255551234" -PhoneNumberType DirectRouting
```

## Call Queues and Auto Attendants

### Auto Attendants
- IVR menu system: "Press 1 for Sales, 2 for Support..."
- Supports: time-of-day routing, holiday schedules, dial-by-name directory
- Can route to: users, call queues, other auto attendants, external numbers, voicemail
- Requires: resource account with Teams Phone Resource Account license (free)

### Call Queues
- Distribute incoming calls to a group of agents
- Routing methods: First In First Out, Round Robin, Longest Idle, Serial
- Features: music on hold, overflow handling, timeout routing, opt-in/opt-out for agents
- Conference mode: faster answer (pre-connects agent) vs transfer mode
- Requires: resource account (may share with auto attendant)

### Resource Accounts
- Every auto attendant and call queue needs a resource account
- License: Teams Phone Resource Account license (free, but required)
- Phone numbers assigned to resource accounts, not directly to AA/CQ
- Managed via Teams Admin Center or PowerShell (`New-CsOnlineApplicationInstance`)

## Emergency Calling

### Dynamic Emergency Calling
- Automatically determines caller's location based on network information
- Location Information Service (LIS): maps subnets, WAPs, switches, ports to civic addresses
- Emergency addresses validated against PSTN carrier's master street address guide

### ELIN (Emergency Location Identification Number)
- Used with Direct Routing for emergency call routing
- Maps emergency locations to ELIN numbers registered with carrier
- Fallback for environments where dynamic location detection is limited

### Configuration
- Emergency calling policies: define notification behavior (who gets notified on 911 call)
- Emergency call routing policies (Direct Routing): define PSTN routing for emergency numbers
- Compliance: FCC/E911 requirements mandate accurate location for emergency calls

## Compliance Recording

### Purpose
- Automatically record calls for regulatory compliance (financial services, healthcare)
- Policy-based recording: applies to specific users, all calls recorded regardless of user action

### Implementation
- Requires certified compliance recording partner solution
- Recording bot joins call transparently; user notified via banner
- Recordings stored in partner's platform (not in Teams/Stream)
- Configured via calling policy: `Set-CsTeamsCallingPolicy -AllowComplianceRecording $true`