## Dataverse

### What Dataverse Actually Is
- Not just a database — it's a **platform** with built-in security, business logic, API, and storage
- Built on SQL Server but abstracted with metadata-driven model
- Every Power Apps Premium and Dynamics 365 plan includes Dataverse
- Tables (formerly entities), columns (formerly fields), relationships, choices (formerly option sets)

### Key Concepts
1. **Tables**: standard (system), custom, virtual (external data without copying)
2. **Relationships**: 1:N, N:N (with intersection table), polymorphic (regarding/customer)
3. **Business rules**: no-code conditional logic on forms (show/hide fields, set defaults, validate)
4. **Calculated/rollup columns**: server-side computations, rollups aggregate child records
5. **Choices (option sets)**: global (reusable) or local (table-specific) picklists

### Security Model
- **Business units** — hierarchical organizational structure
- **Security roles** — define CRUD permissions per table at different scopes (user/BU/parent-child BU/org)
- **Teams** — owner teams (own records) and access teams (share records)
- **Column-level security** — restrict specific columns from specific profiles
- **Record sharing** — ad-hoc access grants beyond role-based access
- **Hierarchy security** — managers can see their reports' data

### Solutions
- **Managed solutions**: deployed to target environments, can't be edited there, cleanly uninstallable
- **Unmanaged solutions**: editable, used in development environments
- **Solution layering**: multiple solutions can customize same component — last one wins (active layer)
- **Segmented solutions**: include only the components you changed, not entire tables
- **Publisher prefix**: use a consistent, unique publisher prefix (not "new_" or "cr...")

### Environment Variables
- Store configuration values that differ between environments (URLs, feature flags, keys)
- Types: text, number, JSON, yes/no, data source
- Set **definition** in solution, set **value** per environment (don't store values in the solution itself)
- **Secret environment variables**: reference Azure Key Vault secrets
- Critical for ALM — avoid hardcoding environment-specific values in apps/flows

### Connection References
- Abstract connections from solutions so they can be rebound per environment
- Solution stores the reference, admin/maker provides the actual connection at import/deployment
- Must be configured for each target environment during deployment
