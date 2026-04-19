## Power Automate

### Cloud Flows vs Desktop Flows

| Aspect | Cloud Flows | Desktop Flows (RPA) |
|---|---|---|
| **Runs on** | Cloud (Power Platform infrastructure) | Local machine (attended or unattended) |
| **Triggers** | Automated, instant (button), scheduled | Called from cloud flows or manually |
| **Best for** | API-based integrations, approvals, notifications | Legacy UI automation, desktop apps, web scraping |
| **Licensing** | Included with most plans (standard connectors) | Requires Power Automate Premium or RPA add-on |
| **Error handling** | Run-after settings, scopes, try-catch pattern | On-error per action, On-block-error for groups |

### Cloud Flow Types
1. **Automated** — triggered by an event (Dataverse record created, email received, file added)
2. **Instant** — triggered manually (button press, Power Apps, Teams message action)
3. **Scheduled** — runs on a recurring schedule (daily, hourly, every 5 minutes)

### Why Flows Fail Silently — Common Causes
1. **Trigger conditions not met** — flow doesn't even start, no run history entry
2. **Connectors missing authentication** — connection expired, user password changed, no re-auth prompt
3. **Apply to Each with no items** — loop runs zero times, succeeds but does nothing
4. **Condition evaluates wrong** — case sensitivity, null values, type mismatches
5. **Throttling** — connector rate limits hit, 429 errors retried and eventually dropped
6. **Delegation warnings in Power Apps** — data not actually sent to flow
7. **Missing error handling** — no scope/try-catch, errors swallowed in parallel branches

### Error Handling Best Practices
1. **Use scopes as try-catch** — "Try" scope for main logic, "Catch" scope with run-after = "Failed"
2. **Configure Run After** on every critical action — specify what happens on failure/timeout/skip
3. **Implement retry policies** — exponential backoff for transient failures (start at 1 min, double each retry)
4. **Terminate on critical errors** — use Terminate action with status "Failed" and meaningful error message
5. **Log errors** — write to SharePoint list, Dataverse table, or Application Insights
6. **Send notifications** — email or Teams message to flow owner on failure
7. **Use `workflow()` function** — get flow run URL for direct links in error notifications
8. **Use `result()` function** with Filter Array in Catch scope to extract specific error details

### Performance and Limits
- **Action limits**: 40K per user/day for premium, 6K for M365-seeded
- **Trigger frequency**: minimum 1 minute for automated triggers (some connectors higher)
- **Run duration**: 30 days max for cloud flows
- **Concurrency**: default parallelism in Apply to Each is 20 (configurable 1-50)
- **Pagination**: some connectors paginate at 256/5000 items — must enable pagination in settings
