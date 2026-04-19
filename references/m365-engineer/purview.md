## Microsoft Purview — Data Governance

### Sensitivity Labels
```
Label Hierarchy:
├── Public
├── Internal
│   ├── Internal Only (no protection, visual marking)
│   └── Internal - Restricted (encryption, no external sharing)
├── Confidential
│   ├── Confidential - All Employees (encryption, all employee access)
│   └── Confidential - Specific People (encryption, named recipients)
└── Highly Confidential
    ├── Highly Confidential - All Employees
    └── Highly Confidential - Specific People

Label capabilities:
├── Visual markings (headers, footers, watermarks)
├── Encryption (Azure Rights Management)
├── Content marking
├── Auto-labeling (client-side or service-side)
├── Container labels (Teams, Groups, Sites — control guest access, sharing, device access)
└── Extend to: Power BI, Azure SQL, Azure Storage, third-party via MIP SDK
```

### Retention Policies & Labels
| Approach | Scope | Use Case |
|----------|-------|----------|
| **Retention policies** | Broad — entire workload (all Exchange, all SharePoint) | Baseline retention across org |
| **Retention labels** | Specific — individual items, folders, libraries | Records management, regulatory holds |
| **Adaptive scopes** | Dynamic — based on user/site attributes | Department-specific retention |
| **Static scopes** | Fixed — specific sites, mailboxes, groups | Targeted retention for known locations |

### DLP (Data Loss Prevention)
- **Locations**: Exchange, SharePoint, OneDrive, Teams, Endpoints, Power BI, on-premises repos
- **Conditions**: sensitive information types (SITs), sensitivity labels, trainable classifiers, file extensions
- **Actions**: block, block with override, notify user, alert admin, encrypt, restrict access
- **Policy tips**: real-time user notifications showing why action was blocked
- **Testing**: always deploy in "test mode with notifications" before enforcement

### Insider Risk Management
- **Signals**: M365 activity, HR connectors (termination dates), Defender for Endpoint, third-party via Graph
- **Policy templates**: data theft by departing users, data leaks, security policy violations, patient data misuse
- **Privacy**: pseudonymized by default, role-based access, audit logs
- **Workflow**: policy triggers alert → triage → investigation → case → action
- **Requires**: E5 or E5 Compliance or Insider Risk Management add-on
