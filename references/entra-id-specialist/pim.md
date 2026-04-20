## Privileged Identity Management (PIM)

### Core Concept

PIM provides just-in-time privileged access — roles are not permanently assigned.
Users hold eligible assignments and activate them only when needed.

### Assignment Types

| Type | Description | When Active |
|------|-------------|-------------|
| **Eligible** | User CAN activate the role | Only after activation request is approved |
| **Active (time-bound)** | User HAS the role now | Until the end time expires |
| **Active (permanent)** | User always has the role | Always — avoid except break-glass |

### Activation Workflow

```
1. User requests activation of eligible role
2. System checks activation requirements:
   ├── MFA verification
   ├── Justification text (why do you need this?)
   ├── Ticket number (link to change request)
   └── Approval (for high-impact roles)
3. Approver reviews and approves/denies
4. Role is active for configured duration (e.g., 4-8 hours)
5. Role automatically deactivates when time expires
6. Full audit trail recorded
```

### Activation Requirements

| Requirement | Purpose | Recommended For |
|-------------|---------|-----------------|
| **MFA** | Verify identity before elevation | All roles |
| **Justification** | Audit trail of why access was needed | All roles |
| **Approval** | Human gate for high-impact roles | Global Admin, Privileged Role Admin |
| **Ticket number** | Link to change management system | All roles (if ITSM integrated) |
| **Conditional Access** | Enforce device/location requirements | Admin roles |

### Access Reviews

Periodic recertification that role assignments are still needed:
- **Frequency**: quarterly for most roles, monthly for Global Admin
- **Reviewers**: self-review, manager, or designated reviewer
- **Scope**: eligible assignments, active assignments, or both
- **Auto-apply**: optionally remove access if not reviewed
- **Reminders**: automatic notifications to reviewers

### Roles That Need PIM Most

| Role | Risk Level | Recommended Settings |
|------|-----------|----------------------|
| Global Administrator | Critical | Approval required, 4-hour max, phishing-resistant MFA |
| Privileged Role Administrator | Critical | Approval required, 4-hour max |
| Exchange Administrator | High | MFA required, 8-hour max, quarterly review |
| SharePoint Administrator | High | MFA required, 8-hour max |
| Application Administrator | High | Approval required (can create credentials) |
| User Administrator | Medium | MFA required, 8-hour max |
| Security Administrator | High | MFA required, 8-hour max, quarterly review |

### Alerts and Monitoring

- Alert on permanent active assignments (should only be break-glass)
- Alert on activations outside business hours
- Alert on failed activation attempts
- Alert on roles activated but never used
- Review activation history monthly
- Track activation frequency per user to identify over-eligible users

### Best Practices

- Convert all permanent admin assignments to eligible (except break-glass)
- Set maximum activation duration: 4-8 hours (never unlimited)
- Require justification for every activation
- Require approval for Global Admin and Privileged Role Admin
- Run access reviews quarterly at minimum
- Use PIM for Azure resource roles too (Owner, Contributor)
