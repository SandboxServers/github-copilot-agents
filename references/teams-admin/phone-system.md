# Teams Admin — Phone System

## Teams Phone System

### PSTN Connectivity Options

| Option | Description | Best For | Infrastructure |
|--------|------------|----------|---------------|
| **Microsoft Calling Plans** | Microsoft is your PSTN carrier, all-in-cloud | Simple orgs, no existing carrier contracts | None — fully cloud |
| **Operator Connect** | Your existing carrier manages PSTN via Microsoft program | Orgs with existing carrier, want managed service | None — carrier manages SBCs |
| **Direct Routing** | Connect your own SBC to Teams Phone | Complex environments, existing PBX, specific carrier requirements | Customer-owned/managed SBC |
| **Teams Phone Mobile** | Use existing mobile carrier number as Teams number | Mobile-first workforces | Carrier participation required |

### Direct Routing Architecture
```
PSTN ←→ SBC ←→ Microsoft 365 (Teams Phone)
                     ↕
               Teams Client

Key components:
- Session Border Controller (SBC): Must be certified for Direct Routing
- Telephony trunk: SIP trunk from PSTN carrier to SBC
- DNS: SBC FQDN must resolve, TLS certificates required
- Firewall: SIP signaling (port 5061 TLS) + media (UDP 49152-53247)
```

### Dial Plans & Normalization
- **Dial plan** = set of normalization rules that translate user-dialed numbers into E.164 format
- **Normalization rules** = regex patterns that match dialed digits and transform them
- Types: Tenant dial plan (assigned to users), Service country dial plan (automatic)
- Common use: 4-digit extension dialing → translate to full E.164 number
- PowerShell: `New-CsOnlineVoiceRoute`, `Set-CsOnlineVoiceRoutingPolicy`, `Grant-CsOnlineVoiceRoutingPolicy`
