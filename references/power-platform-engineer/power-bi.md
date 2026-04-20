## Power BI

### Data Modeling
- **Star schema** — fact tables surrounded by dimension tables, avoid snowflake
- Avoid bidirectional relationships unless required (security and performance impact)
- Define measures explicitly with DAX — don't rely on implicit measures
- Always create a dedicated date dimension table, mark as date table
- Use calculated columns sparingly — prefer Power Query transformations or measures

### DAX Essentials
- **CALCULATE** — modifies filter context, the most important DAX function
- **Time intelligence** — TOTALYTD, SAMEPERIODLASTYEAR, DATEADD for period comparisons
- **Iterator vs aggregator** — SUMX (row-by-row) vs SUM (column), choose based on need
- **Variables** — VAR/RETURN for readability and performance (evaluated once)
- **ALL/ALLEXCEPT** — remove filters for percentage-of-total patterns
- **RELATED/RELATEDTABLE** — navigate relationships in formulas

### Storage Modes

| Mode | Data Location | Performance | Freshness |
|---|---|---|---|
| **Import** | In-memory in Power BI | Fastest queries | Stale between refreshes |
| **DirectQuery** | Stays in source system | Slower (live queries) | Always current |
| **Composite** | Mix of both per table | Balanced | Varies by table |

- Import is default and best for most scenarios — schedule refresh as needed
- DirectQuery for real-time requirements or datasets too large to import
- Composite models let you import dimensions and DirectQuery large fact tables

### Row-Level Security (RLS)
- Define roles with DAX filter expressions in Power BI Desktop
- Filters rows — cannot restrict columns, tables, or measures
- Map Entra ID security groups to roles (not individual users)
- RLS applies only to Viewer role — Admins/Members/Contributors bypass it
- Multiple roles are additive (union, not intersection)
- Dynamic RLS with `USERPRINCIPALNAME()` for per-user filtering
- Apply RLS to dimension tables not fact tables for better performance

### Deployment Pipelines
- Promote content through Dev → Test → Production stages
- Compare content across stages to identify differences before deployment
- Parameterized deployment rules for data source changes per environment
- Supports semantic models, reports, dashboards, and dataflows

### Dataflows
- Reusable ETL/data prep — output to Dataverse or Azure Data Lake
- Shared across multiple reports and semantic models
- Schedule refresh independently from reports
- Back up definitions — deleted dataflows cannot be recovered

### Paginated Reports
- Pixel-perfect, printable reports designed for export (PDF, Excel, Word)
- Best for operational reports, invoices, statements with many pages
- Built in Power BI Report Builder (separate tool from Desktop)
- Render at query time — not limited by in-memory model size

### Workspaces and Apps
- Workspaces organize content by team, project, or domain
- Apps package workspace content for read-only distribution to consumers
- Use workspace roles: Admin, Member, Contributor, Viewer

### Capacity and Licensing
- **Pro** — basic sharing, publish to workspaces, 1 GB model size limit
- **Premium Per User (PPU)** — Pro + larger models, deployment pipelines, AI features
- **Premium capacity** — dedicated resources, unlimited viewers, XMLA endpoint, 400 GB models
- Fabric capacity replaces standalone Premium capacity for new deployments

### Refresh and Gateway
- Scheduled refresh: up to 8/day (Pro), 48/day (Premium)
- Incremental refresh for large datasets — only refresh new/changed data
- On-premises data gateway required for on-prem sources (SQL Server, files, Oracle)
- Gateway cluster for high availability and load balancing
