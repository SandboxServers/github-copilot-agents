## Security Review Document Template

### Structure
1. **Executive Summary** — overall risk assessment (Critical/High/Medium/Low), key findings count
2. **Scope** — what was reviewed (architecture, code, configuration, compliance framework)
3. **Findings** — each finding with:
   - Severity (Critical/High/Medium/Low/Informational)
   - Category (Identity, Network, Data, Compliance, Configuration)
   - Description of the issue
   - Risk impact
   - Remediation with specific Azure configuration
   - Effort estimate (hours/days)
4. **Compliance Status** — per-framework compliance assessment
5. **Recommendations** — prioritized action items with owners and timelines
6. **Sign-off** — explicit approval or rejection with conditions

### Severity Classification
| Severity | Definition | SLA |
|---|---|---|
| **Critical** | Active exploitation risk, data breach imminent, regulatory violation | Block deployment. Fix immediately. |
| **High** | Significant vulnerability, compliance gap, major misconfiguration | Fix before production. Max 5 business days. |
| **Medium** | Defense-in-depth gap, best practice violation, minor exposure | Fix within 30 days. Track in backlog. |
| **Low** | Hardening opportunity, minor configuration improvement | Fix within 90 days or accept risk. |
| **Informational** | Observation, suggestion for improvement, no immediate risk | No SLA. Consider in next iteration. |

## Common Security Anti-Patterns

### Identity
- Permanent Global Admin assignments (use PIM)
- Service principals with subscription-level Owner
- Shared credentials or connection strings in app settings
- No MFA for admin accounts
- Break-glass accounts not tested or monitored

### Network
- Public endpoints on PaaS services without IP restrictions
- SSH/RDP open to 0.0.0.0/0
- No WAF on internet-facing applications
- Service endpoints used where private endpoints are available
- No network segmentation between environments (dev/prod share VNet)

### Data
- Unencrypted data at rest (using default without verifying)
- TLS 1.0/1.1 still enabled
- No data classification — all data treated the same
- Connection strings in code or environment variables (use Key Vault references)
- No DLP policies on sensitive data stores

### Operations
- No diagnostic settings on critical resources
- Defender for Cloud recommendations unaddressed
- No incident response plan or untested plan
- Security alerts going to unmonitored mailbox
- No access reviews — stale permissions accumulate
