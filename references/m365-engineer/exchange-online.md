## Exchange Online — Mail Flow

### Email Authentication Stack
```
SPF (Sender Policy Framework)
├── TXT record in DNS: v=spf1 include:spf.protection.outlook.com -all
├── Lists authorized sending IPs/services for your domain
├── Only ONE SPF record per domain (combine all authorized senders)
└── Hard fail (-all) recommended, soft fail (~all) during migration

DKIM (DomainKeys Identified Mail)
├── Cryptographic signature in email headers
├── Enable in Defender portal → Email authentication → DKIM
├── Two CNAME records per domain (selector1, selector2)
└── Proves email content wasn't modified in transit

DMARC (Domain-based Message Authentication, Reporting, and Conformance)
├── TXT record: v=DMARC1; p=reject; rua=mailto:dmarc@contoso.com
├── Tells receivers what to do when SPF/DKIM fail
├── Policies: none (monitor) → quarantine → reject (progressive deployment)
└── Aggregate reports (rua) show who's sending as your domain

Deployment order: SPF first → DKIM → DMARC (p=none) → monitor → DMARC (p=reject)
```

### Connectors

| Connector Type | Direction | Use Case |
|---------------|-----------|----------|
| **Partner** | Inbound/Outbound | Third-party email services, security appliances |
| **From your org's email server** | Inbound | Hybrid mail flow from on-prem Exchange |
| **To your org's email server** | Outbound | Route mail to on-prem for processing |

### Transport Rules (Mail Flow Rules)
- **Conditions**: sender, recipient, subject, body, headers, attachments, message properties
- **Actions**: modify headers, add disclaimers, redirect, BCC, reject, encrypt, set SCL
- **Exceptions**: carve-outs from conditions
- **Priority**: lower number = higher priority, first match wins
- **Testing**: always test with "test this rule" or audit mode before enforcing
- **Limits**: up to 300 rules per org, 8KB per rule, 20 actions per rule

### Hybrid Mail Flow
```
Scenario 1: MX → M365 (recommended for new deployments)
├── Internet mail delivered to Exchange Online Protection (EOP)
├── EOP filters spam/malware → delivers to Exchange Online mailbox
└── On-prem routing via connector for on-prem mailboxes

Scenario 2: MX → On-premises
├── Internet mail delivered to on-prem Exchange/third-party filter
├── On-prem routes to Exchange Online via connector
└── Requires Enhanced Filtering for Connectors for proper protection

Scenario 3: MX → Third-party service → M365
├── Internet mail to third-party spam filter first
├── Third-party routes to Exchange Online
├── Enable Enhanced Filtering for Connectors (skip listing)
└── Important: include spf.protection.outlook.com in SPF record
```
