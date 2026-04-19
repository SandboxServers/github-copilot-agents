## Migration Playbooks

### From On-Premises Exchange
```
Phase 1: Prepare
├── Hybrid Configuration Wizard (HCW) → establish connectivity
├── Configure Entra Connect for identity sync
├── Verify mail flow (MX, SPF, connectors)
└── Pilot: move IT mailboxes first

Phase 2: Migrate
├── Batch migration (cutover for <150 mailboxes, staged for larger)
├── Migration endpoint: Exchange Web Services (EWS) or REST
├── Schedule batches: move 100-500 mailboxes per batch
├── Monitor: migration dashboard, move request stats
└── User communication: outlook profile auto-update

Phase 3: Complete
├── Update MX records to point to Exchange Online
├── Decommission hybrid server (or keep for attribute management)
├── Reconfigure send connectors, shared mailboxes, distribution groups
└── Validate: mail flow, free/busy, public folders
```

### From Google Workspace
```
Phase 1: Prepare
├── Add and verify domain in M365 admin center
├── Provision user accounts (Entra Connect or manual)
├── Configure Migration Manager for Google
├── Grant service account access to Google Workspace

Phase 2: Migrate
├── Gmail → Exchange Online (email, calendar, contacts)
├── Google Drive → OneDrive/SharePoint (files, permissions)
├── Google Shared Drives → SharePoint team sites
├── Google Sites → SharePoint pages (limited, often manual rebuild)
└── Google Chat history → not directly migrated (export first)

Phase 3: Cut Over
├── Update MX records → M365
├── Update SPF/DKIM/DMARC for new platform
├── Redirect Google Drive links (temporary forwarding)
├── User training: Outlook, OneDrive, Teams adoption
└── Decommission Google Workspace licenses (after retention period)
```
