## Power Platform Anti-Patterns

### No DLP Policies
- Without DLP, any maker can connect any connector to any data source
- Results in connector sprawl — Dataverse connected to random SaaS apps in same flow
- Production data accessible through unvetted third-party connectors
- **Fix**: implement tenant-wide DLP baseline, restrict premium connectors by default

### Everything in the Default Environment
- Default environment is accessible to every user in the tenant — cannot restrict access
- Production apps in default environment = zero isolation, zero governance
- Cleanup and management become impossible when hundreds of apps pile up
- **Fix**: lock down default environment, create dedicated environments per purpose

### No ALM (Manual Solution Management)
- Moving apps by manual export/import with no version control or change tracking
- No way to roll back broken deployments — overwrite and pray
- Configuration drift between environments (different connection strings, missing components)
- No audit trail of who changed what and when
- **Fix**: adopt solution-based development, use Pipelines for Power Platform or DevOps tooling

### Ignoring Delegation Limits
- Non-delegable formulas silently return only 500/2000 records from data source
- App appears to work in dev (small dataset) but fails silently in production (large dataset)
- Users see incomplete data without any error message — dangerous for business decisions
- Common culprit: `Search()`, `CountRows()`, `in` operator on non-delegable sources
- **Fix**: check delegation warnings, restructure formulas, use Dataverse views for complex queries

### No Error Handling in Flows
- Flows fail silently — no one knows until a business process breaks downstream
- Apply to Each with zero items succeeds but does nothing — no notification
- Parallel branches swallow errors in sibling branches
- Connection expirations cause flows to stop running with no alert
- **Fix**: implement try-catch scope pattern, log errors, send failure notifications

### Premium Connectors Without License Planning
- Maker adds one SQL connector or Dataverse lookup — entire app becomes premium
- Every user of the app now needs premium license — budget surprise
- Shadow premium usage accumulates, discovered only during license audit
- **Fix**: audit connectors before building, educate makers on licensing implications, use DLP to prevent

### No Governance (Shadow Apps)
- Makers build apps without oversight — no inventory, no standards, no reviews
- Duplicate apps solving same problem built by different teams
- Sensitive data accessed through unreviewed apps with no security review
- Apps become orphaned when maker leaves the organization — no documentation, no owner
- **Fix**: deploy CoE Starter Kit for inventory, establish review process, assign ownership

### Building Complex Apps That Should Be Coded
- Forcing Power Platform to handle scenarios it was never designed for
- Complex custom UI (animations, real-time collaboration, intricate layouts)
- High-performance requirements (sub-100ms, millions of records in view)
- Apps with >50,000 users where licensing exceeds custom development cost
- Complex offline scenarios beyond Power Apps offline capabilities
- **Fix**: use Power Platform for what it's good at, hand off complex requirements to pro dev teams

### Other Common Mistakes
- **Hardcoded values** instead of environment variables — breaks on deployment
- **No naming conventions** — Button1, Flow1, Table1 make maintenance impossible
- **Overloading a single app** with too many screens (>15) — split into multiple apps
- **Not using solutions** — building directly in the environment loses all ALM capability
- **Ignoring solution checker warnings** — technical debt accumulates silently
- **Single-maker dependency** — only one person understands the app, no documentation
