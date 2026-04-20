## Power Platform Governance

### Environment Strategy
```
Environments
├── Default Environment
│   ├── Every user has access automatically
│   ├── Minimize use — restrict with DLP policies
│   └── Cannot be deleted or renamed
├── Production Environments
│   ├── Department-specific (HR, Finance, IT)
│   ├── Project-specific for major initiatives
│   └── Controlled access via security groups
├── Sandbox Environments
│   ├── Testing and training
│   ├── Can be reset to clean state
│   └── Isolate experimental work from production
└── Developer Environments
    ├── Personal development space
    ├── Auto-deleted after period of inactivity
    └── One per user, self-service provisioning
```

### Key Governance Decisions
- Who can create environments — restrict to IT or admins
- Environment request process — CoE toolkit or IT service management
- DLP policy assignment per environment
- Environment lifecycle — decommission unused environments

### DLP Policies (Data Loss Prevention for Power Platform)
- **Connector classification**: Business / Non-Business / Blocked
- **Business connectors**: allowed to share data with each other (SharePoint, Outlook, Dataverse)
- **Non-Business connectors**: share data with each other but NOT with Business connectors
- **Blocked connectors**: cannot be used in the environment at all
- **Scope**: tenant-wide baseline + environment-specific overrides
- **Strategy**: block risky connectors tenant-wide, selectively allow per environment

### Connector Group Best Practices
- Start restrictive — block HTTP, custom connectors, and premium connectors by default
- Promote connectors to Business group only when justified by use case
- Review new connectors quarterly — Microsoft adds connectors regularly
- Document exceptions and approvals for non-standard connector access

### CoE (Center of Excellence) Starter Kit
- **Inventory**: discover all apps, flows, connectors, makers across environments
- **Governance**: compliance processes, archive inactive resources, environment request workflows
- **Nurture**: training resources, maker assessment, community management
- **Audit**: tenant hygiene, monitor DLP violations, connector usage tracking
- **Key value**: enables citizen development with governance guardrails

### Managed Environments
- **Sharing limits**: restrict how broadly canvas apps can be shared
- **Solution checker enforcement**: require solution checker to pass before import
- **Maker onboarding**: display custom welcome message and policy links
- **Usage insights**: built-in analytics without CoE toolkit
- **Data policies**: additional DLP enforcement options
- **Licensing**: requires Power Apps, Power Automate, or Dynamics premium license

### Citizen Developer Enablement with Guardrails
- Provide training paths and certification tracks
- Establish maker community with champions network
- Define app quality standards — naming conventions, error handling, documentation
- Require solution-aware development for production apps
- Implement app lifecycle management — dev → test → prod promotion

### Power Platform Admin Center
- Manage environments, capacity, DLP policies, analytics
- Monitor tenant-wide resource consumption
- View maker activity and app usage metrics
- Configure tenant isolation and data policies
- Manage Power Platform request capacity and add-ons
