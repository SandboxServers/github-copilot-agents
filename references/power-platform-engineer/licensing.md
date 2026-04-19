## Licensing

### The Premium vs Standard Trap

**Standard connectors** (included with M365):
- SharePoint, Outlook, Teams, OneDrive, Excel, Planner, Approvals
- Good for basic productivity automation

**Premium connectors** (require Power Apps Premium or Power Automate Premium):
- Dataverse, SQL Server, Azure services, Dynamics 365, HTTP, custom connectors, on-prem gateway
- **This is the trap**: the moment someone adds a Dataverse table or SQL connection, the app becomes premium
- **Every user of a premium app needs a premium license** — not just the maker

### License Plans
| Plan | Price (approx.) | What You Get |
|---|---|---|
| **M365 (seeded)** | Included | Standard connectors only, 6K actions/day, basic canvas apps |
| **Power Apps per app** | ~$5/user/app/month | One app per user, premium connectors, Dataverse |
| **Power Apps Premium** | ~$20/user/month | Unlimited apps, premium connectors, Dataverse, 40K actions/day |
| **Power Automate Premium** | ~$15/user/month | Unlimited cloud flows, premium connectors, attended RPA |
| **Power Automate Process** | ~$150/bot/month | Unattended RPA (one bot) |
| **Pay-as-you-go** | Per active user/month | Azure subscription billing, no upfront commitment |

### Design Around Licensing
1. **Audit connector usage early** — before building, know which connectors you need and which licenses they require
2. **Keep M365 apps on standard connectors** — don't accidentally make a SharePoint app premium by adding SQL
3. **Use per-app plans** for narrow use cases — cheaper than per-user when <4 apps per user
4. **Power Automate seeded with Power Apps** — Power Apps Premium includes Power Automate for associated flows
5. **Pay-as-you-go** for seasonal/variable usage — good for apps used by many people infrequently
