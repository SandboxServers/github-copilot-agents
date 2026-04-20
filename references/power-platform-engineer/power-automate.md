## Power Automate

### Cloud Flow Types
1. **Automated flows** — triggered by events (Dataverse record created, email received, file added)
2. **Instant flows** — triggered manually (button press, Power Apps call, Teams message action)
3. **Scheduled flows** — run on recurring schedule (daily, hourly, every 5 minutes)

### Desktop Flows (RPA)
- Automate legacy desktop applications and web interfaces via UI recording
- Attended (user watches) or unattended (runs on dedicated machine without user)
- Called from cloud flows to bridge cloud-to-desktop automation
- Requires Power Automate Premium or dedicated RPA add-on license
- Best for: legacy systems without APIs, data entry automation, web scraping

### Business Process Flows
- Guided stage-based processes displayed as a bar at the top of model-driven apps
- Define stages with steps, conditions, and branching logic
- Track where each record is in the process lifecycle
- Enforce data entry requirements at each stage before advancement
- Integrate with cloud flows to trigger automation at stage transitions

### Connectors

| Type | Examples | Licensing |
|---|---|---|
| **Standard** | SharePoint, Outlook, Teams, OneDrive, Excel | Included with M365 |
| **Premium** | Dataverse, SQL Server, HTTP, Azure, custom connectors | Requires premium license |
| **Custom** | Any REST API wrapped as a connector | Premium license required |

- Adding one premium connector makes the entire flow premium-licensed
- Custom connectors require OpenAPI definition or build from scratch
- On-premises data gateway extends connectors to on-prem data sources

### Error Handling

**Try-Catch with Scopes**
- Wrap main logic in a "Try" scope action
- Add "Catch" scope configured with run-after = "Failed, TimedOut"
- Use `result('TryScopeName')` in Catch to extract error details
- Add "Finally" scope with run-after = "Succeeded, Failed" for cleanup

**Configure Run-After**
- Set on every critical action: what happens on success, failure, timeout, skip
- Without run-after configuration, a failed action stops the entire flow

**Retry Policies**
- Configure on HTTP and connector actions: fixed interval or exponential backoff
- Default: 4 retries with exponential backoff
- Set custom intervals for known transient failure patterns

### Expressions and Functions
- `workflow()` — get flow run URL for error notification links
- `triggerBody()` / `triggerOutputs()` — access trigger data
- `body()` / `outputs()` — reference action results
- `concat()`, `substring()`, `replace()` — string manipulation
- `formatDateTime()`, `addDays()` — date operations
- `json()`, `xml()` — data format conversion

### Solution-Aware Flows
- Build flows inside solutions for proper ALM and environment promotion
- Use connection references instead of hardcoded connections
- Use environment variables for configuration that changes per environment
- Child flows enable reusable flow logic (called from parent flows)

### Best Practices
1. **Avoid unnecessary triggers** — use trigger conditions to filter before the flow runs
2. **Handle errors** — every production flow needs try-catch pattern
3. **Use child flows** — extract reusable logic, keep parent flows readable
4. **Limit Apply to Each concurrency** — default 20, reduce for rate-limited connectors
5. **Enable pagination** — some connectors return only 256 items by default
6. **Log important events** — write to Dataverse or SharePoint for audit trails
7. **Terminate with meaningful messages** — include error details and flow run URL
