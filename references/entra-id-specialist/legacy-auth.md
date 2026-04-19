## Legacy Auth Protocols — The Kill List

### What Needs to Die
| Protocol | Status | Risk | Migration Path |
|----------|--------|------|---------------|
| **Basic Auth (Exchange)** | Deprecated Oct 2022 | No MFA possible, credential stuffing target | Modern Auth (OAuth 2.0) |
| **Legacy Exchange ActiveSync** | Block via CA | No MFA, no conditional access | Outlook mobile with modern auth |
| **POP3/IMAP** | Block via CA | No MFA, credentials in clear text (without TLS) | Outlook/modern clients |
| **SMTP AUTH** | Needed for some printers/scanners | Limited MFA support | App passwords or OAuth for supported clients |
| **Older Office clients** | Office 2013 and earlier | NTLM/Basic auth | Upgrade to modern Office |

### How to Block
```
Conditional Access Policy:
- Users: All users (exclude break-glass)
- Cloud apps: All cloud apps
- Conditions → Client apps: 
  ✅ Exchange ActiveSync clients
  ✅ Other clients
- Grant: Block access

Important: Test with report-only mode first!
Check sign-in logs for legacy auth usage before blocking.
Filter: Client app = Exchange ActiveSync, Other clients, IMAP, MAPI, POP, SMTP
```
