## SharePoint Online — Architecture

### Site Types & Hierarchy
```
SharePoint Tenant
├── Home Site (intranet landing page)
│
├── Hub Sites (organizational connectors)
│   ├── HR Hub
│   │   ├── Benefits Team Site
│   │   ├── Recruiting Team Site
│   │   └── HR Policies Communication Site
│   ├── IT Hub
│   │   ├── Helpdesk Team Site
│   │   └── IT News Communication Site
│   └── Marketing Hub
│       ├── Brand Team Site
│       └── Campaigns Communication Site
│
├── Team Sites (collaboration — M365 Group-connected)
│   ├── Private group membership
│   ├── Shared mailbox, calendar, Planner
│   └── Teams channel sites
│
└── Communication Sites (broadcast — no M365 Group)
    ├── Organization-wide news
    ├── Departmental portals
    └── Project showcases
```

### Hub Sites
- **Purpose**: group related sites, share navigation and theme, roll up content (news, events, activity)
- **Limit**: 2,000 hub associations per tenant
- **Best practice**: organize by function (HR, IT, Finance) or region, not deep hierarchy
- **Permissions**: hub association doesn't change site permissions — content is security-trimmed
- **Navigation**: hub owners control shared navigation bar across all associated sites

### Content Types
- Defined at the **Content Type Hub** (site collection), published to subscribing sites
- Include: metadata columns, document templates, workflows
- Use for: consistent metadata across sites (e.g., "Project Document" type with Project Name, Phase, Owner columns)
- Modern: SharePoint Syntex / Microsoft Syntex for AI-powered content understanding

### Search Configuration
- **Crawled properties** → mapped to **Managed properties** → available in search queries
- Customization via: result sources, query rules, result types, search schema
- Modern search: Microsoft Search in Microsoft 365 — unified across SharePoint, OneDrive, Teams, Outlook
- **Search verticals**: custom tabs in search results (People, Sites, Files, custom)
- **Bookmarks & Q&A**: admin-curated answers for common queries

### Migration Strategies

| Source | Tool | Key Considerations |
|--------|------|-------------------|
| **SharePoint Server** | SharePoint Migration Tool (SPMT), Migration Manager | Assess with SMAT, remediate before migration, plan hub structure |
| **File shares** | Migration Manager | Folder → site/library mapping, permission mapping |
| **Google Workspace** | Migration Manager for Google | Drive → OneDrive/SharePoint, permissions, shared drives |
| **Box/Dropbox** | Mover (now Migration Manager) | API-based migration, plan for throttling |
| **Other tenants** | Cross-tenant migration | Requires Entra ID cross-tenant access policies |
