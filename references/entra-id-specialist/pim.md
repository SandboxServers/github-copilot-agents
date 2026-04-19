## Privileged Identity Management (PIM)

### Core Concepts
- **Eligible assignment**: User CAN activate the role, but doesn't have it until they do
- **Active assignment**: User HAS the role right now (time-bound or permanent)
- **Activation**: User requests to use an eligible role — may require justification, MFA, approval
- **Time-bound**: All activations have an expiration (default 8 hours, configurable)

### PIM Best Practices
1. **Convert permanent → eligible** for all admin roles (except break-glass)
2. **Require justification** for all activations (creates audit trail)
3. **Require approval** for high-impact roles (Global Admin, Exchange Admin, SharePoint Admin)
4. **Set max activation duration** — 4-8 hours typical, never unlimited
5. **Require MFA** on activation (or better: phishing-resistant MFA)
6. **Access reviews** — quarterly for all privileged roles, monthly for Global Admin
7. **Alert on** permanent assignments, activations outside business hours, failed activations

### Roles That Need PIM Most
| Role | Risk Level | Recommended Settings |
|------|-----------|---------------------|
| Global Administrator | Critical | Approval required, 4-hour max, phishing-resistant MFA, monthly review |
| Privileged Role Administrator | Critical | Approval required, 4-hour max |
| Exchange Administrator | High | MFA required, 8-hour max, quarterly review |
| SharePoint Administrator | High | MFA required, 8-hour max |
| User Administrator | Medium | MFA required, 8-hour max |
| Application Administrator | High | Approval required (can create app credentials) |
| Cloud Application Administrator | High | Approval required |
