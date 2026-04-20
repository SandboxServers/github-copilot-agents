## Power Platform Anti-Patterns (Broad)

> Complements [pp-anti-patterns.md](pp-anti-patterns.md) with broader organizational and architectural mistakes.

### No Environment Strategy
- Everything lives in the default environment — accessible to every tenant user
- No isolation between dev/test/prod; changes hit production immediately
- Cannot restrict maker access or apply environment-specific DLP policies
- Cleanup is impossible when hundreds of orphaned apps pile up
- **Fix**: rename default to "Personal Productivity," create dedicated environments per purpose
- Ref: <https://learn.microsoft.com/power-platform/guidance/adoption/environment-strategy>

### No DLP Policies
- Premium connectors (SQL, HTTP, custom) accessible to every maker in every environment
- Sensitive data flows through unvetted third-party connectors in the same flow
- No separation of business vs non-business vs blocked connector groups
- **Fix**: implement tenant-wide DLP baseline, restrict premium by default, allow exceptions per environment

### Canvas Apps with Hardcoded Connection Strings
- Connection strings embedded in formulas or app properties instead of environment variables
- Breaks on deployment to test/prod — manual edits required every time
- Credentials exposed to anyone who can view the app source
- **Fix**: use connection references + environment variables for all environment-specific config

### No ALM Strategy
- No solutions — components built directly in the environment
- No environment variables or connection references
- Manual export/import with no version control, no audit trail
- Cannot roll back broken deployments or track who changed what
- **Fix**: solution-based development, Pipelines for Power Platform or Azure DevOps integration

### Direct Dataverse Queries Instead of Views/FetchXML
- Client-side filtering of large datasets instead of server-side views
- Pulling entire tables into collections then filtering locally
- Triggers Dataverse query anti-pattern errors (`PerformanceLeadingWildCard`, `LargeAmountOfAttributes`)
- **Fix**: use saved views, FetchXML, or delegable filter expressions
- Ref: <https://learn.microsoft.com/power-apps/developer/data-platform/query-antipatterns>

### Power Automate Flows with No Error Handling
- Flows fail silently — no one knows until downstream processes break
- Parallel branches swallow errors in sibling branches
- Connection expirations cause flows to stop with no alert
- **Fix**: implement try-catch scope pattern, configure run-after on failed/timed-out, send failure notifications

### Too Many Approvals in a Single Flow
- Chaining 10+ approval actions hits connector throttling limits
- Approval connector has daily action limits per user (tied to license tier)
- Long-running approval flows risk hitting the 30-day timeout
- **Fix**: break into sub-flows, use parallel approvals where possible, consider custom approval tables

### Shared Connections Owned by Individual Users
- Production flows use connections tied to a specific employee's credentials
- When that person leaves or password changes, all dependent flows break simultaneously
- No visibility into which flows depend on which connections
- **Fix**: use service accounts or service principals for production connections, document connection ownership

### No Naming Conventions
- `Button1`, `Flow 1`, `Table1` — impossible to maintain or audit
- Makers cannot distinguish their flows from others in shared environments
- CoE Starter Kit compliance reports become meaningless
- **Fix**: enforce naming standard (e.g., `DEPT-App-Purpose`, `DEPT-Flow-Purpose`), publish in maker onboarding

### Model-Driven App with 50+ Forms
- Navigation becomes a maze — users can't find the right form
- Performance degrades with excessive form loading and rendering
- Business rules multiply across forms creating maintenance nightmares
- **Fix**: consolidate forms by audience, use business rules to show/hide sections, consider custom pages

### Power BI Reports Connecting Directly to Transactional Systems
- Import mode against production OLTP databases locks tables during refresh
- DirectQuery sends every user interaction as a live query — impacts transactional performance
- No aggregation layer means slow visuals and frustrated users
- **Fix**: use a dedicated analytics database/lakehouse, implement incremental refresh, use aggregations

### No Monitoring of Flow Failures
- Failed flows accumulate silently — discovered only when business processes break
- No alerting, no dashboard, no regular review of flow run history
- Orphaned flows continue to fail indefinitely with no owner investigating
- **Fix**: deploy CoE Starter Kit for centralized monitoring, configure flow failure alerts, assign flow owners
- Ref: <https://learn.microsoft.com/power-platform/guidance/coe/limitations>
