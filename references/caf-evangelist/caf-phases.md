## CAF Methodology Overview (7 Phases)

```
FOUNDATIONAL (Sequential)              OPERATIONAL (Parallel, ongoing)
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    ┌──────────┐  ┌──────────┐  ┌──────────┐
│ Strategy  │→│   Plan   │→│  Ready   │→│  Adopt   │    │  Govern  │  │  Secure  │  │  Manage  │
│           │  │          │  │          │  │          │    │          │  │          │  │          │
│ Business  │  │ Ops model│  │ Tenant   │  │ Migrate  │    │ Risk     │  │ Controls │  │ RAMP     │
│ drivers → │  │ Skills   │  │ Platform │  │ Modernize│    │ Policy   │  │ Identity │  │ Ready    │
│ Cloud     │  │ Migration│  │ Landing  │  │ Cloud-   │    │ Compliance│ │ Network  │  │ Administer│
│ outcomes  │  │ plan     │  │ zones    │  │ native   │    │ Cost     │  │ Data     │  │ Monitor  │
└──────────┘  └──────────┘  └──────────┘  └──────────┘    └──────────┘  └──────────┘  │ Protect  │
                                                                                       └──────────┘
```

### Phase Details

| Phase | Key Outcome | Key Activities | Common Mistakes |
|-------|------------|----------------|-----------------|
| **Strategy** | Cloud adoption aligned to business goals | Map business drivers to cloud outcomes, define motivations (migrate, innovate, optimize), build business case | Skipping strategy → adoption without direction. "We're moving to cloud because everyone else is." |
| **Plan** | Actionable cloud adoption plan | Define operating model, assess skills, create migration plan, estimate costs, document workload inventory | Planning in isolation without cross-team input. Over-planning without starting. |
| **Ready** | Azure environment prepared for workloads | Azure purchasing, tenant setup, platform landing zone, application landing zones | Deploying workloads before landing zone is ready. "We'll add governance later." |
| **Adopt** | Workloads running in Azure | Migrate (8 Rs), modernize, or build cloud-native | Treating every workload the same (all rehost, or all rearchitect). |
| **Govern** | Controlled cloud environment | Assess risks, define policies, enforce with Azure Policy, manage costs | Governance as afterthought → shadow IT, cost overruns, compliance gaps. |
| **Secure** | Protected workloads | Security controls, identity, network security, data protection | Security bolted on at the end instead of built in from Strategy phase. |
| **Manage** | Optimized operations | RAMP (Ready, Administer, Monitor, Protect), operational excellence | No monitoring → finding issues when users complain. |
