## Dataverse

### Table Design

**Table Types**
- **Standard tables** — built-in system tables (Account, Contact, Activity)
- **Custom tables** — your business-specific tables (prefix with publisher prefix)
- **Virtual tables** — map to external data sources without copying data into Dataverse
- **Elastic tables** — Azure Cosmos DB-backed for high-volume, low-latency scenarios

**Column Types**
- Text (single-line, multi-line, rich text), number (whole, decimal, float, currency)
- Date/time, yes/no (boolean), choice (option set), lookup, file, image
- Calculated columns — server-side formulas evaluated on read
- Rollup columns — aggregate child records (sum, count, min, max, avg)
- Formula columns — Power Fx expressions, real-time calculation

### Relationships
- **1:N (one-to-many)** — parent-child, lookup column on the child table
- **N:N (many-to-many)** — creates an intersection table automatically
- **Polymorphic** — single lookup references multiple table types (Customer, Regarding)
- Cascade behaviors: assign, share, unshare, reparent, delete — configure per relationship

### Business Rules
- No-code conditional logic applied to forms (model-driven and some canvas)
- Show/hide fields, set field values, set required level, show error messages
- Scope: entity (server-side) or specific form (client-side only)
- No looping or complex logic — use plug-ins or Power Automate for that

### Security Model

**Business Units**
- Hierarchical organizational structure — every user belongs to one business unit
- Root BU at the top, child BUs nested underneath
- Security roles are assigned within a business unit context

**Security Roles**
- Define CRUD+A (Create, Read, Write, Delete, Append, Assign, Share) per table
- Scope levels: User, Business Unit, Parent-Child BU, Organization
- Assign multiple roles — permissions are additive (most permissive wins)

**Teams**
- Owner teams — own records, have security roles assigned directly
- Access teams — dynamic membership, share records without owning them
- Entra ID group teams — membership synced from Entra ID security groups

**Field-Level Security**
- Restrict read/update/create on specific columns regardless of table-level permissions
- Create field security profiles, add columns, assign users/teams
- Use for sensitive data: SSN, salary, confidential notes

### Views, Charts, and Dashboards
- **Views** — saved queries (system views managed in solutions, personal views by users)
- **Charts** — visual representations tied to views
- **Dashboards** — combine charts, views, iframes, Power BI tiles in one screen
- All configurable declaratively in model-driven app designer

### Dataverse for Teams
- Simplified Dataverse included with M365 Teams license (no premium license)
- Limited to 1M rows, 2 GB per environment, max 5 custom tables
- Apps built in Teams, accessed in Teams — not available outside Teams
- Can upgrade to full Dataverse if needs outgrow Teams limitations

### Environment Strategy with Dataverse
- Each environment gets its own Dataverse instance (isolated data and schema)
- Dev environment: unmanaged solutions, active development
- Test/staging: managed solutions, validation and UAT
- Production: managed solutions only, strict change control
- Dataverse capacity shared across environments in tenant — monitor usage
