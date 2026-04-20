## Microsoft Purview — Data Governance

### Sensitivity Labels
```
Label Hierarchy:
├── Public
├── Internal
│   ├── Internal Only (visual marking, no encryption)
│   └── Internal - Restricted (encryption, block external sharing)
├── Confidential
│   ├── Confidential - All Employees (encryption, all employee access)
│   └── Confidential - Specific People (encryption, named recipients only)
└── Highly Confidential
    ├── Highly Confidential - All Employees
    └── Highly Confidential - Specific People

Label scope and capabilities:
├── Files and emails — visual markings, encryption, access restrictions
├── Groups, sites, Teams — control guest access, sharing, unmanaged device access
├── Meetings — watermarks, prevent copying chat, end-to-end encryption
├── Schematized data assets — Azure SQL, Azure Storage, Power BI
└── Auto-labeling — client-side recommendations and service-side automatic application
```

### Auto-Labeling
- **Client-side**: label recommendations appear in Office apps, user accepts or dismisses
- **Service-side**: automatic labeling without user interaction (requires E5 or E5 Compliance)
- Based on: sensitive information types (SITs), trainable classifiers, exact data match
- Simulation mode: test auto-labeling policies before enforcement
- Priority: manual label overrides auto-label; higher-priority label cannot be downgraded by auto-label

### Retention Policies vs Retention Labels

| Feature | Retention Policies | Retention Labels |
|---------|-------------------|------------------|
| Scope | Broad — entire workload or locations | Specific — individual items |
| Application | Automatic to all content in scope | Manual, auto-apply, or default label |
| Use case | Baseline retention across org | Records management, regulatory holds |
| Disposition | Silent deletion at end of period | Disposition review, proof of deletion |

**Supported workloads**: Exchange, SharePoint, OneDrive, Teams (chat and channel), Viva Engage (Yammer)

**Scope types**:
- **Adaptive scopes**: dynamic — based on user or site attributes (department, location)
- **Static scopes**: fixed — specific sites, mailboxes, groups

### DLP (Data Loss Prevention)
- **Locations**: Exchange, SharePoint, OneDrive, Teams, Endpoints, Power BI, on-premises repositories
- **Conditions**: sensitive information types, sensitivity labels, trainable classifiers, file extensions
- **Actions**: block, block with override, notify user, alert admin, encrypt, restrict access
- **Policy tips**: real-time user notifications explaining why action was blocked
- **Testing**: deploy in test mode with notifications before enforcement
- **Endpoint DLP**: extends protection to Windows and macOS devices — copy to USB, print, upload

### Insider Risk Management
- **Signals**: M365 activity, HR connectors (termination, resignation), Defender for Endpoint, third-party via Graph
- **Policy templates**: data theft by departing users, data leaks, security policy violations, patient data misuse
- **Privacy**: pseudonymized by default, role-based access, full audit logging
- **Workflow**: policy trigger → alert → triage → investigation → case → action
- **Requires**: E5, E5 Compliance, or Insider Risk Management add-on

### eDiscovery

| Tier | Capabilities |
|------|-------------|
| Content search | Search across Exchange, SharePoint, OneDrive, Teams |
| eDiscovery Standard | Cases, holds, exports, search by custodian |
| eDiscovery Premium | Custodian management, review sets, analytics, conversation threading, near-duplicate detection |

### Data Lifecycle Management
- Retention policies and labels for automated data retention and deletion
- Records management for regulatory and legal requirements
- Disposition review before permanent deletion of declared records
- File plan: import and manage retention labels at scale with descriptors
