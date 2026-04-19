## Governance

### Environment Strategy
```
Default Environment          → LOCK DOWN. No production apps. DLP restricts all premium connectors.
Personal Productivity        → Sandbox for experimentation. Auto-cleanup policy (90 days inactive).
Shared Development           → Shared dev with CoE oversight. Standard DLP.
Test/Staging                 → Pre-production validation. Mirror prod DLP.
Production                   → Managed Environment. Strict DLP. Approval required for deployment.
```

- **Lock the default environment** — every user has access to it, it's the #1 governance gap
- Use **Managed Environments** for production — maker welcome content, sharing limits, solution checker enforcement
- **Environment groups** — apply rules/DLP policies across groups of environments at once
- Restrict environment creation to admins (Power Platform admin center → Tenant settings)

### DLP (Data Loss Prevention) Policies
- Classify connectors into **Business** (can talk to each other), **Non-Business** (can talk to each other), and **Blocked**
- Apply at tenant level (baseline) and environment level (override for specific needs)
- **Start restrictive, loosen with justification** — easier than tightening later
- Key principle: production Dataverse should never be connectable to random SaaS apps in the same flow
- **Regularly audit new connectors** — Microsoft adds connectors frequently, they may not fit your policy

### CoE (Center of Excellence) Starter Kit
- Collection of Power Apps, Power Automate flows, Power BI reports for governance and monitoring
- **Core module**: inventory of all apps, flows, makers, environments in tenant
- **Governance module**: compliance processes, inactivity notifications, cleanup workflows
- **Nurture module**: maker community engagement, training tracking, app showcases
- Built on Dataverse — deployed as solutions
- **Not a replacement for CoE people/process** — it's tooling that supports human governance

### Citizen Developer Enablement (Without Chaos)
1. **Training before access** — maker onboarding with org-specific guidelines
2. **Welcome content** in Managed Environments — show policies when makers first open Power Apps
3. **Templates** — provide pre-built, approved patterns (with org branding, correct data sources)
4. **Local expert teams** — power users who coach citizen devs and review apps before production
5. **Promotion process** — dev → staging → production requires review/approval
6. **Auto-cleanup** — inactive apps/flows flagged after 60-90 days, deleted after grace period
