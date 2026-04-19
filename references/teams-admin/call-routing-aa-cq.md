# Teams Admin — Call Routing (Auto Attendants & Call Queues)

### Call Routing Configuration
```
Voice Routing Policy (assigned to user)
  └─ PSTN Usages (containers)
       └─ Voice Routes (number patterns + SBC priority)
            └─ Online PSTN Gateways (SBC endpoints)

Example flow:
User dials +1-425-555-1234
→ Matches pattern +1425XXXXXXX in Voice Route "Seattle"  
→ Route tries SBC sbc1.contoso.com (priority 1)
→ If unavailable, tries sbc2.contoso.com (priority 2)
→ SBCs within same route are tried in random order
```

### Auto Attendants & Call Queues
| Feature | Auto Attendant | Call Queue |
|---------|---------------|------------|
| Purpose | IVR menu system (press 1 for sales...) | Distribute calls to group of agents |
| Routing | Menu options, dial by name, operator | First in first out, round robin, longest idle, serial |
| Resource account | Required (with phone number) | Required (may share with AA) |
| Music on hold | Customizable | Customizable |
| Overflow/timeout | Redirect to voicemail, user, queue, AA | Redirect after max wait time or max queue size |
| Nesting | AA can route to AA or CQ | CQ can overflow to AA |
