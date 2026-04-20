## Licensing Model

### Per-User vs Per-App Licensing

| Model | Price (approx.) | Best For |
|---|---|---|
| **Power Apps per user** | ~$20/user/month | Users who need unlimited apps |
| **Power Apps per app** | ~$5/app/user/month | Users who only need 1-2 specific apps |
| **Power Automate per user** | ~$15/user/month | Users who need unlimited cloud flows |
| **Power Automate per flow** | ~$100/month (5 flows) | Flows shared by many users (no per-user cost) |

- Per-user: unlimited apps/flows for that user — better when users access many apps
- Per-app: license specific apps individually — cheaper when users need few apps
- Per-flow: license the flow itself, not the users — ideal for org-wide automated processes

### Premium Connector Licensing Trigger
- Using any premium connector in an app or flow requires premium licensing
- **Every user** of the app/flow needs the premium license — not just the maker
- Standard connectors: SharePoint, Outlook, Teams, OneDrive, Excel, Planner
- Premium connectors: Dataverse, SQL Server, HTTP, Azure services, custom connectors
- One premium connector in the app = entire app requires premium licensing
- Audit connector usage before building — avoid accidental premium escalation

### M365 Seeded Rights
- Included with M365 E3/E5, Business Basic/Standard/Premium licenses
- **Standard connectors only** — no Dataverse, SQL, HTTP, or custom connectors
- Limited to basic canvas apps and cloud flows within M365 context
- 6,000 Power Automate actions per user per day (vs 40K for premium)
- Cannot use apps outside of M365 context (no standalone Power Apps)
- Good for simple SharePoint forms, Teams automations, Outlook workflows

### Dataverse Capacity
- Dataverse storage allocated per tenant based on license mix
- **Database capacity**: 1 GB base + per-license increments
- **File capacity**: 2 GB base + per-license increments for attachments and files
- **Log capacity**: 2 GB base + per-license increments for audit logs
- Monitor capacity in Power Platform admin center — overages block new data
- Add-on capacity packs available for purchase

### Pay-As-You-Go
- Bill through Azure subscription instead of pre-purchased licenses
- Charged per active user per app per month (meters through Azure)
- No upfront commitment — good for seasonal or variable usage patterns
- Useful for apps with many users who access infrequently
- Link Power Platform environment to Azure subscription to enable
- Includes premium connectors and Dataverse access

### Licensing Design Decisions
1. **Audit connectors first** — know which are premium before building
2. **Keep M365 apps on standard connectors** — don't accidentally trigger premium
3. **Per-app for narrow use cases** — cheaper than per-user when users need <4 apps
4. **Power Apps Premium includes Power Automate** — for flows associated with the app
5. **Per-flow for org-wide processes** — avoids licensing every user who benefits from automation
6. **Pay-as-you-go for experimentation** — test adoption before committing to annual licenses
