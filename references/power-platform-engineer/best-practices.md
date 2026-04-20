## Power Platform Best Practices

### Environment Strategy
- Minimum three environments: Development → Test → Production
- Provision sandbox environments for experimentation and training
- Rename default environment to "Personal Productivity" — restrict its use
- Restrict production/trial environment creation to admins only
- Use Entra security groups to control environment access
- Ref: <https://learn.microsoft.com/power-platform/guidance/adoption/environment-strategy>

### DLP Policies
- Classify connectors into Business / Non-Business / Blocked groups
- Apply a tenant-wide baseline DLP policy — restrict premium connectors by default
- Create environment-specific overrides only when justified with a documented exception
- Block HTTP and custom connectors in environments where they aren't needed
- Review DLP policies quarterly as new connectors are added to the platform

### ALM: Solutions, Environment Variables, Connection References, Pipelines
- Always develop inside solutions — never build directly in the default solution
- Use environment variables for all configuration that differs between environments
- Use connection references so connections can be swapped at deployment time
- Automate deployments with Pipelines for Power Platform or Azure DevOps/GitHub Actions
- Store solution exports in source control as the single source of truth
- Ref: <https://learn.microsoft.com/power-platform/alm/environment-strategy-alm>

### Canvas App Performance
- Minimize `App.OnStart` — move initialization to `App.StartScreen` or `Screen.OnVisible`
- Use `Concurrent()` for independent data loads to parallelize network calls
- Limit gallery items to what's visible — avoid loading 500+ rows into a gallery
- Avoid nested galleries beyond one level — each nesting multiplies render cost
- Cache frequently used lookups in collections rather than repeated calls
- Check delegation warnings aggressively — treat them as errors, not suggestions
- Ref: <https://learn.microsoft.com/power-apps/maker/canvas-apps/create-performant-apps-overview>

### Dataverse: Table Design, Relationships, Security Roles
- Design tables with proper primary name columns and meaningful alternate keys
- Use 1:N and N:N relationships instead of storing foreign keys as text fields
- Apply field-level security for sensitive columns (SSN, salary, PII)
- Assign security roles at the team level rather than per-user for easier management
- Use business units to scope data access when organizational hierarchy matters
- Limit the number of columns per table — wide tables degrade query performance

### Power Automate: Error Handling
- Use the try-catch scope pattern: main actions in a "Try" scope, error handling in a "Catch" scope
- Configure run-after on the Catch scope for `has failed` and `has timed out`
- Log failures to a Dataverse "Flow Errors" table or send to a Teams channel/email
- Set reasonable timeout values on HTTP and approval actions — don't rely on defaults
- Use `terminateWithError` action to surface meaningful error messages to callers
- Test error paths explicitly — don't assume the happy path is the only path

### Power BI: Star Schema, DAX, Incremental Refresh
- Model data using star schema — fact tables surrounded by dimension tables
- Avoid bi-directional relationships unless absolutely required (performance and ambiguity risks)
- Use calculated columns sparingly — prefer calculated measures (DAX) for aggregations
- Enable incremental refresh for large datasets to reduce refresh time and resource usage
- Use aggregation tables for DirectQuery models to improve visual response times
- Implement row-level security (RLS) for multi-tenant or role-based data access

### Governance: CoE Starter Kit
- Deploy the Center of Excellence (CoE) Starter Kit for tenant-wide visibility
- Use the CoE dashboard to monitor app/flow inventory, orphaned resources, and connector usage
- Establish a maker onboarding process with training, guardrails, and a welcome kit
- Define an app/flow retirement policy — archive or delete resources with no owner or recent usage
- Conduct quarterly governance reviews covering DLP compliance, license usage, and maker activity
- Ref: <https://learn.microsoft.com/power-platform/guidance/coe/limitations>

### Naming Conventions
- Prefix apps: `DEPT-AppName` (e.g., `HR-LeaveRequest`, `FIN-ExpenseTracker`)
- Prefix flows: `DEPT-FlowPurpose` (e.g., `HR-OnboardingApproval`)
- Prefix Dataverse tables: `prefix_TableName` using publisher prefix
- Name environment variables descriptively: `AppConfig_APIBaseURL`, `AppConfig_ApprovalEmail`
- Document naming standards in a shared wiki — enforce via maker onboarding and CoE compliance rules

### Security and Compliance
- Use Entra Conditional Access policies to restrict Power Platform access from unmanaged devices
- Enable audit logging in Dataverse for tables containing sensitive data
- Apply field-level security profiles for PII and financial columns
- Review connector permissions quarterly — revoke access to unused or risky connectors
- Use managed environments for production workloads to enforce solution checker and sharing limits

### Monitoring and Observability
- Set up email/Teams alerts for flow failures — don't rely on makers checking run history
- Use Power Platform admin analytics for tenant-wide usage and adoption metrics
- Monitor Dataverse storage consumption — file, log, and database storage have separate limits
- Track API request usage against entitlement limits to avoid throttling surprises
- Review the CoE Starter Kit dashboards weekly for orphaned apps, expired connections, and compliance drift
