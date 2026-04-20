## Shadow IT Governance

### The Shadow IT Problem

Shadow IT happens when official tools are too locked down, too slow to provision, or users don't know they exist. Common forms:

- Unsanctioned cloud apps (file sharing, project management, messaging)
- Personal email forwarding of business data
- Ungoverned Power Apps and Power Automate flows
- Third-party AI tools processing company data
- Teams and M365 Groups created without naming or governance policies

### Discovery with Defender for Cloud Apps

Defender for Cloud Apps (CASB) is the primary discovery tool:

1. **Cloud Discovery**: Analyze firewall/proxy logs or use the Defender for Endpoint agent to identify all cloud apps in use
2. **App catalog**: 30,000+ apps rated by risk across 90+ risk factors (compliance, security, legal)
3. **Risk scoring**: Each discovered app gets a risk score — prioritize investigation by risk level and usage volume
4. **Usage patterns**: See which users, how much data, upload vs download patterns

### Governance Actions

For each discovered app, choose one of three actions:

| Action | When to Use | Implementation |
|--------|------------|----------------|
| **Sanction** | App is approved and meets security/compliance requirements | Add to sanctioned apps, configure SSO through Entra, apply Conditional Access |
| **Monitor** | App is under review or low-risk but not officially approved | Tag as monitored, set alerts for increased usage or sensitive data transfer |
| **Block** | App is high-risk, redundant with approved tools, or violates policy | Block via Defender for Cloud Apps, create user notification explaining approved alternative |

### Power Platform DLP Policies

Power Platform introduces citizen development — users building apps and automations without IT involvement. DLP policies control what connectors can be used together:

- **Business data group**: Connectors approved for business data (SharePoint, Outlook, Dataverse, Teams)
- **Non-business data group**: Personal or external connectors (Twitter, Gmail, personal OneDrive)
- **Blocked group**: Connectors that cannot be used at all in the environment

**Key rules:**
- Connectors in different groups cannot be used in the same flow or app
- Apply DLP policies at the tenant level as a baseline, then per-environment for exceptions
- Default environment gets the strictest policy — it's where all users land by default
- Premium connectors require premium licensing — DLP alone doesn't control cost

### Citizen Developer Enablement with Guardrails

The goal is not to prevent citizen development — it is to channel it safely:

1. **Managed environments**: Create dedicated Power Platform environments for departments with appropriate DLP policies
2. **Maker welcome content**: Custom guidance that appears when users open Power Apps — link to governance policies and training
3. **Environment routing**: Remove users from the default environment and assign them to managed environments
4. **App registration governance**: Require admin consent for new app registrations in Entra — prevent ungoverned API access
5. **Regular review**: Quarterly audit of Power Platform apps and flows — identify high-usage, high-risk, or abandoned solutions
6. **Center of Excellence (CoE) kit**: Microsoft's governance toolkit for Power Platform — deploy to gain visibility into tenant-wide usage

### Discovery Cadence

| Frequency | Action |
|-----------|--------|
| **Continuous** | Defender for Cloud Apps monitoring, automated alerts on new high-risk apps |
| **Weekly** | Review new app discoveries, triage sanctioned/monitored/blocked |
| **Monthly** | Power Platform usage review, environment audit, new connector evaluation |
| **Quarterly** | Full shadow IT report, DLP policy review, citizen developer program assessment |
