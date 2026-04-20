## Exchange Online

### Email Authentication Stack
```
SPF (Sender Policy Framework)
├── TXT record: v=spf1 include:spf.protection.outlook.com -all
├── Authorizes sending IPs and services for your domain
├── One SPF record per domain — combine all authorized senders
└── Hard fail (-all) recommended; soft fail (~all) during migration only

DKIM (DomainKeys Identified Mail)
├── Cryptographic signature in email headers
├── Enable in Defender portal → Email authentication → DKIM
├── Two CNAME records per domain (selector1, selector2)
└── Validates email content was not modified in transit

DMARC (Domain-based Message Authentication, Reporting, Conformance)
├── TXT record: v=DMARC1; p=reject; rua=mailto:dmarc@contoso.com
├── Instructs receivers on SPF/DKIM failure handling
├── Progressive deployment: none → quarantine → reject
└── Aggregate reports (rua) reveal unauthorized senders

Deployment order: SPF → DKIM → DMARC (p=none) → monitor → DMARC (p=reject)
```

### Mail Flow Connectors

| Connector Type | Direction | Use Case |
|---------------|-----------|----------|
| Partner | Inbound/Outbound | Third-party email services, security appliances |
| From your org's email server | Inbound | Hybrid mail flow from on-prem Exchange |
| To your org's email server | Outbound | Route mail to on-prem for processing |

### Transport Rules (Mail Flow Rules)
- **Conditions**: sender, recipient, subject, body, headers, attachments, message properties
- **Actions**: modify headers, add disclaimers, redirect, BCC, reject, encrypt, set SCL
- **Exceptions**: carve-outs from conditions
- **Priority**: lower number = higher priority, first match wins
- **Testing**: use audit mode before enforcing
- **Limits**: 300 rules per org, 8 KB per rule, 20 actions per rule

### Mailbox Types

| Type | Purpose | License Required |
|------|---------|------------------|
| User mailbox | Primary mailbox for licensed users | Yes |
| Shared mailbox | Shared access (< 50 GB), no license needed | No (under 50 GB) |
| Room mailbox | Conference room booking | No |
| Equipment mailbox | Projectors, vehicles, other resources | No |

### Groups

| Group Type | Mail-Enabled | Use Case |
|-----------|-------------|----------|
| Microsoft 365 Group | Yes | Teams, SharePoint, Planner, shared mailbox |
| Distribution list | Yes | Email distribution only |
| Dynamic distribution list | Yes | Auto-membership based on attributes |
| Mail-enabled security group | Yes | Email distribution + permission assignment |

### Hybrid Exchange
- **Exchange Hybrid Wizard (HCW)**: configures connectors, OAuth, certificates, org relationships
- **Minimal hybrid**: free/busy and mail flow only — no mailbox moves
- **Full hybrid**: bidirectional mailbox migration, shared namespace, unified GAL
- **Requires**: at least one on-prem Exchange server (or management server for Exchange 2019+)

### Migration Methods

| Method | Best For | Details |
|--------|---------|--------|
| Cutover | < 150 mailboxes, no hybrid | All mailboxes migrated at once |
| Staged | Large orgs, Exchange 2003/2007 | Batches, coexistence period |
| Hybrid | Exchange 2010+ | Ongoing coexistence, bidirectional moves |
| IMAP | Non-Exchange sources | Email only (no calendar, contacts) |
| PST import | Archive ingestion | Network upload or drive shipping |

### Retention (Messaging Records Management)
- **MRM retention policies**: assigned to mailboxes, contain retention tags
- **Retention tags**: default policy tag (DPT), retention policy tag (RPT), personal tags
- **Actions**: delete, move to archive, permanently delete
- **Note**: MRM is legacy — Microsoft recommends Purview retention policies for new deployments
