## SharePoint Online — Architecture

### Hub Sites
- Group related sites under a shared navigation and theme
- Roll up content across associated sites — news, events, activity
- Limit: 2,000 hub associations per tenant
- Organize by function (HR, IT, Finance) or region, not deep hierarchy
- Hub association does not change site permissions — content is security-trimmed
- Hub owners control the shared navigation bar for all associated sites
- Register hub sites via SharePoint admin center or PowerShell

### Content Types
- Defined at Content Type Hub, published to subscribing sites
- Include metadata columns, document templates, workflows
- Enable consistent metadata across sites (e.g., Project Document type with Project Name, Phase, Owner)
- Modern content understanding via SharePoint Premium (formerly Syntex)
- Content type syndication — central management, local site consumption

### Search
- **Crawled properties** mapped to **managed properties** for query use
- Customization: result sources, query rules, result types, search schema
- Microsoft Search in Microsoft 365 — unified across SharePoint, OneDrive, Teams, Outlook
- **Search verticals**: custom tabs in results (People, Sites, Files, custom)
- **Bookmarks and Q&A**: admin-curated answers for common queries
- **Managed properties**: define sortable, queryable, retrievable, refinable attributes
- Continuous crawl keeps index near real-time

### Site Templates
- **Team site**: collaboration, M365 Group-connected, private membership
- **Communication site**: broadcast content, no M365 Group, wider audience
- **Custom site templates**: apply via site scripts and site designs (JSON-based automation)
- **Site provisioning**: PnP Provisioning Engine for complex templates with content types, lists, pages

### Storage Management
- Tenant storage pool shared across all sites (default 1 TB + 10 GB per licensed user)
- Per-site storage limits configurable in SharePoint admin center
- Automatic or manual storage management modes
- Monitor storage via admin center reports or PowerShell
- Version history consumes storage — configure version limits per library

### External Sharing Settings

| Level | Description |
|-------|-------------|
| Anyone | Anonymous sharing links (most permissive) |
| New and existing guests | Guests must sign in, invitations sent |
| Existing guests | Only guests already in the directory |
| Only people in your organization | No external sharing |

- Tenant-level sets the maximum permissiveness
- Site-level can be equal to or more restrictive than tenant
- OneDrive sharing inherits from or is more restrictive than SharePoint setting

### Versioning
- Major versions: enabled by default in document libraries
- Minor versions (drafts): optional, useful for publishing approval workflows
- Version limits: configurable per library — balance history vs storage
- Automatic version history management trims older versions to save storage

### Information Architecture Best Practices
- Flat hierarchy — avoid deeply nested subsites (use hub sites instead)
- Metadata over folders — use columns and views rather than folder trees
- Consistent naming conventions across sites and libraries
- Plan navigation: global nav (hub), local nav (site), footer links
- Governance: site lifecycle policies — creation requests, expiration, archival
