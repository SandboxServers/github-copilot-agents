## Citizen Developer Enablement

### What Citizen Developers Are
- Business users who build apps and automations without traditional coding skills
- Closest to the business problems — understand processes and data better than IT
- Use Power Apps, Power Automate, and Power BI to solve their own workflow problems
- Not professional developers — need guardrails, not gatekeeping

### Training Paths
- Microsoft Learn paths for Power Apps, Power Automate, Power BI fundamentals
- Organization-specific onboarding: your DLP policies, naming conventions, data sources
- Hands-on workshops with real business scenarios (not just tutorials)
- Certification paths: PL-100 (Power Platform App Maker), PL-900 (Fundamentals)
- Progressive skill building: basic apps → complex flows → Dataverse modeling

### Maker Community
- Internal maker community (Teams channel, Yammer group, or SharePoint hub)
- Regular show-and-tell sessions for makers to demo their solutions
- Peer mentoring between experienced and new makers
- Shared component libraries and template galleries
- Recognition program for high-impact citizen-built solutions

### Best Practices Templates
- Pre-built app templates with org branding, correct data sources, security patterns
- Flow templates for common scenarios (approvals, notifications, data sync)
- Naming convention guides and checklists
- Architecture decision templates (when to use canvas vs model-driven)
- Data source selection guide (SharePoint list vs Dataverse vs Excel)

### Review Process for Production Apps
1. **Self-assessment checklist** — maker reviews against standards before submission
2. **Peer review** — another maker or local expert reviews app design and logic
3. **Technical review** — IT/CoE reviews security, data access, performance, licensing impact
4. **Solution checker** — automated scan for common issues and best practice violations
5. **Approval gate** — formal sign-off before promoting to production environment

### Guardrails
- **DLP policies** — prevent citizen devs from connecting unauthorized data sources
- **Environment policies** — citizen devs work in sandbox environments, not production
- **Sharing limits** — Managed Environments restrict how widely apps can be shared
- **Connector restrictions** — block HTTP and custom connectors in citizen dev environments
- **Auto-cleanup** — flag inactive apps after 60-90 days, delete after grace period

### Center of Excellence Model
- Dedicated CoE team: platform admin, governance lead, maker enablement lead
- CoE Starter Kit deployed for inventory, analytics, and compliance automation
- Regular governance reviews of apps, flows, and connector usage
- Escalation path for citizen devs who need premium capabilities or complex integrations
- Annual platform health assessment and policy refresh

### App Lifecycle for Citizen-Built Apps
- **Ideation** — maker identifies problem, checks if solution already exists
- **Build** — develop in personal or sandbox environment with proper solution packaging
- **Review** — submit for peer and technical review before production
- **Deploy** — promote through environments (dev → test → production)
- **Maintain** — maker remains responsible for updates and support
- **Retire** — inactive apps flagged, owners notified, graceful deprecation

### When to Graduate to Pro Developer
- App requires complex custom connectors or Azure integration
- Performance requirements exceed Power Platform capabilities
- Need full unit testing, CI/CD, and source control workflows
- Customer-facing application where brand experience is critical
- Over 50,000 active users (licensing cost exceeds custom development)
- Logic complexity exceeds what Power Fx and flows can handle cleanly
- Hand off to pro dev team with documentation of business requirements and data model
