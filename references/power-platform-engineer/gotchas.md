## Power Platform Gotchas

### Dataverse: Solution-Aware vs Non-Solution-Aware Components
- Not all Dataverse components are solution-aware (e.g., some admin settings, certain security configs)
- Non-solution-aware components cannot be transported between environments via solutions
- Building outside of solutions means components land in the default solution — hard to isolate
- Always check the solution layer before assuming a component will migrate cleanly

### Power Automate: Trigger Conditions vs Filter Conditions
- **Trigger conditions** prevent the flow from running at all (evaluated before the run starts)
- **Filter conditions** (Filter Array action) run after the trigger fires — the flow run still counts
- Trigger conditions use OData-style expressions, not Power Fx — different syntax than the rest of the platform
- Using filter actions instead of trigger conditions wastes API calls and flow run quota

### Canvas Apps: Delegation Limits (500/2000 Row Default)
- Non-delegable formulas silently return only the first 500 records (configurable up to 2000)
- App appears to work in dev with small datasets — fails silently in production
- `Search()`, `CountRows()`, `Distinct()`, `in` operator are common non-delegable culprits
- No error message — users see incomplete data without knowing it
- **Mitigation**: check delegation warnings, restructure formulas, use Dataverse views
- Ref: <https://learn.microsoft.com/power-apps/maker/canvas-apps/delegation-overview>

### Power BI: Import vs DirectQuery Performance Tradeoffs
- **Import mode**: fast queries, scheduled refresh, but data can be stale between refreshes
- **DirectQuery**: real-time data, but every visual interaction generates a live query to the source
- DirectQuery performance depends entirely on the source system — no Power BI optimization possible
- Composite models (mixing both) add complexity but can balance freshness vs performance
- Large Import models hit the 1GB/dataset limit (10GB with Premium) without incremental refresh

### Dataverse: Calculated vs Rollup Columns — Different Refresh Timing
- **Calculated columns**: evaluate in real-time when the record is retrieved
- **Rollup columns**: refresh asynchronously on a system job schedule (default: every hour, max 12 hours)
- Rollup values can appear stale immediately after child record changes
- Cannot force an immediate rollup refresh from a canvas app — only via API or on-demand recalc button
- Formula columns (newer) replace calculated columns with Power Fx but have their own limitations

### Managed Solutions: Cannot Edit Managed Components in Target
- Managed solution components are read-only in the target environment by design
- To customize, you must create an unmanaged layer on top — which then must be maintained separately
- Deleting a managed solution removes all its components and data from the target environment
- Managed solution updates can fail if there are unmanaged customizations on the same component
- Always test solution import in a non-production environment first

### Power Automate: Concurrent Flow Runs Behave Differently by Trigger
- **Automated triggers** (Dataverse, SharePoint) can run multiple instances concurrently by default
- **Recurrence/Schedule triggers** queue subsequent runs if the previous run hasn't finished
- Concurrency control settings are per-flow and affect throughput vs ordering guarantees
- Turning on concurrency control changes loop behavior — `Apply to each` processes items in parallel
- Default concurrency is 50 for Apply to Each — lowering it may be needed for rate-limited APIs

### Licensing: Premium Connectors Require Per-User or Per-App Plan
- Adding a single premium connector (SQL, HTTP with Azure AD, Dataverse in certain contexts) makes the entire app premium
- Every user of that app needs a Power Apps Premium license — budget surprise at scale
- Some connectors are "standard" in one product but "premium" in another (e.g., Dataverse for Teams vs standalone)
- License enforcement can be delayed — apps work initially but get blocked during compliance sweeps

### Canvas Apps: Performance Issues with Large Galleries and Complex Formulas
- Galleries with 100+ visible items and complex per-item formulas cause severe rendering lag
- `CountRows(Filter(...))` inside gallery item templates triggers repeated non-delegable queries
- Nested galleries multiply the performance cost — avoid nesting beyond one level
- **Mitigation**: limit visible items, use `Concurrent()` for independent data loads, minimize OnStart logic
- Ref: <https://learn.microsoft.com/power-apps/maker/canvas-apps/create-performant-apps-overview>

### Power Automate Desktop: Runs Under User Context
- Desktop flows execute under the signed-in user's Windows session — not a service account by default
- Unattended mode requires a specific machine registration and Premium license
- If the user's session is locked/logged out, the attended desktop flow fails
- Machine groups and unattended bots require careful capacity planning and additional licensing

### Dataverse: Business Units Affect Security Role Inheritance
- Security roles are scoped to business units — a role assigned at BU-A doesn't apply to records in BU-B
- Moving a user between business units can silently strip their security role assignments
- Teams (owner teams and access teams) have their own BU scope, adding another layer of complexity
- The root business unit cannot be deleted or renamed — plan your hierarchy carefully from the start

### Power Automate: 30-Day Flow Run Duration Limit
- Cloud flows have a maximum run duration of 30 days — after that, the run is cancelled
- Long-running approval workflows that wait for human response can hit this silently
- Workaround: use a Dataverse "pending approval" table and a scheduled flow to check/retry
- Do-until loops also have a default 60-iteration and 1-hour timeout — must be configured explicitly

### Dataverse: Choice Column Values Are Not Portable
- Choice (option set) integer values can differ between environments if created manually
- Global choices shared across tables can cause conflicts during solution import if modified in target
- Always define choices inside solutions and avoid editing integer values after initial creation

### Power BI: Row-Level Security Does Not Apply to Workspace Admins
- Users with Admin or Member role in a Power BI workspace bypass RLS entirely
- This means developers and admins always see all data — they cannot test RLS from their own account
- Use "View as Role" in Power BI Desktop to test, but it's not a substitute for end-user validation

### Canvas Apps: Media Files Inflate App Size
- Images and media embedded directly in a canvas app increase the .msapp package size
- Large app packages cause slow load times, especially on mobile devices
- **Mitigation**: host media in SharePoint or Blob Storage and reference by URL instead of embedding
