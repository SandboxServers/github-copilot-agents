## Power Platform Governance

### Environment Strategy
```
Environments
├── Default Environment (every user gets access — minimize use)
├── Production Environments (governed, controlled access)
│   ├── Department-specific (HR, Finance, IT)
│   └── Project-specific
├── Sandbox/Dev Environments (testing, training)
└── Developer Environments (personal dev, auto-deleted when inactive)

Key decisions:
- Who can create environments? (restrict to IT/admins)
- How are environments requested? (process via CoE toolkit or ServiceNow)
- What DLP policies apply to each environment?
- How are environments decommissioned?
```

### DLP Policies (Data Loss Prevention for Power Platform)
- **Connector classification**: Business / Non-Business / Blocked
- **Business connectors**: allowed to share data with each other (e.g., SharePoint + Outlook)
- **Non-Business connectors**: can share data with each other but NOT with Business connectors
- **Blocked connectors**: cannot be used at all in the environment
- **Scope**: tenant-wide (baseline) + environment-specific (override)
- **Strategy**: baseline DLP blocks risky connectors tenant-wide, per-environment DLP loosens for specific needs

### CoE (Center of Excellence) Starter Kit
- **Inventory components**: discover all apps, flows, connectors, makers across environments
- **Governance components**: compliance processes, archive inactive resources, environment request workflow
- **Nurture components**: training resources, maker assessment, community management
- **Audit components**: tenant hygiene, monitor DLP violations, connector usage
- **Key insight**: enables citizen development while maintaining governance guardrails
