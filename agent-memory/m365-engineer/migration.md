## Migration Planning

### Migration Sources and Approaches

| Source | Primary Method | Key Tool |
|--------|---------------|----------|
| On-premises Exchange | Hybrid migration | Exchange Hybrid Wizard |
| Google Workspace | API-based migration | Migration Manager for Google |
| Another M365 tenant | Cross-tenant migration | Microsoft cross-tenant migration tooling |
| IMAP servers | IMAP migration | Exchange admin center |
| File shares | File migration | Migration Manager |
| SharePoint Server | Site migration | SharePoint Migration Tool (SPMT), Migration Manager |
| Box, Dropbox | Cloud-to-cloud | Migration Manager (formerly Mover) |

### Assessment Phase
- **Inventory users**: active, inactive, shared mailboxes, room mailboxes, distribution lists
- **Mailbox sizing**: identify large mailboxes (> 50 GB), plan batch priority
- **Data inventory**: email, calendar, contacts, OneDrive/file shares, SharePoint sites
- **Permission audit**: shared mailbox delegates, calendar permissions, site permissions
- **Application dependencies**: LOB apps using SMTP relay, calendar APIs, SharePoint APIs
- **Identity readiness**: Entra Connect sync configured, UPN matching, password hash sync or passthrough auth

### Migration Tools

| Tool | Scope | Notes |
|------|-------|-------|
| Exchange Hybrid Wizard | Exchange mailbox moves | Requires on-prem Exchange server |
| SharePoint Migration Tool (SPMT) | SharePoint Server → SPO | Desktop tool, supports incremental |
| Migration Manager | Files, SharePoint, Google | Cloud-based, agent-based for on-prem |
| ShareGate (third-party) | SharePoint, Teams, OneDrive | GUI-based, permission mapping, pre/post reports |
| BitTitan MigrationWiz (third-party) | Exchange, files, archives | SaaS-based, supports many source platforms |
| PST import (network upload) | Archive ingestion | Upload to Azure blob, import via compliance center |

### Cutover Planning
- **MX record switch**: schedule during low-traffic window, reduce DNS TTL to 300 seconds 48 hours before
- **DNS updates**: MX, SPF, DKIM CNAMEs, DMARC, Autodiscover CNAME
- **Rollback plan**: document original DNS records, keep source accessible for 72+ hours
- **Validation**: send test emails, verify free/busy, check calendar sharing, test mobile devices
- **Coexistence period**: plan for split routing if migrating in batches

### User Communication Plan
- **Pre-migration**: announce timeline, what to expect, self-service preparation steps
- **During migration**: status updates, helpdesk availability, known issues list
- **Post-migration**: training resources, new feature highlights, feedback channel
- **Support ramp**: increase helpdesk staffing during and after cutover
- **Quick reference guides**: Outlook profile setup, OneDrive sync, Teams basics
