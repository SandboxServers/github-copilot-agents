# Security Review Process

This agent has **VETO POWER**. Nothing ships to production without security sign-off.

## Review Checklist

Every architecture, deployment, and configuration change is reviewed against these categories:

### Identity
- [ ] Managed identity used instead of secrets/connection strings?
- [ ] Least privilege RBAC — roles assigned at narrowest scope?
- [ ] No shared keys or shared access signatures for production?
- [ ] Conditional access policies enforcing MFA?
- [ ] PIM configured for privileged roles?
- [ ] Service principal credentials have expiry and rotation plan?

### Network
- [ ] Private endpoints deployed for all PaaS services?
- [ ] NSGs on all subnets with deny-by-default rules?
- [ ] No public endpoints in production (or IP-restricted if required)?
- [ ] WAF on all internet-facing web applications?
- [ ] DDoS Protection Standard on VNets with public IPs?
- [ ] Azure Bastion or JIT for management access (no direct SSH/RDP)?

### Data
- [ ] Encryption at rest — platform-managed keys minimum, CMK for sensitive data?
- [ ] Encryption in transit — TLS 1.2+ enforced, HTTP disabled?
- [ ] Data classification applied — controls match classification level?
- [ ] Key management via Key Vault with RBAC model?

### Secrets
- [ ] All secrets stored in Key Vault (not in code, config, or pipeline variables)?
- [ ] No secrets in source code or appsettings files?
- [ ] Rotation policy defined and automated where possible?
- [ ] Managed identity preferred over secrets for all service-to-service auth?

### Monitoring
- [ ] Defender for Cloud enabled with appropriate plans?
- [ ] Diagnostic settings configured on all resources?
- [ ] Alert rules for security events (failed logins, privilege escalation, unusual access)?
- [ ] Logs sent to Log Analytics workspace with appropriate retention?

### Compliance
- [ ] Regulatory requirements identified and documented?
- [ ] Azure Policy assignments in place for required compliance frameworks?
- [ ] Audit trail available (Activity Log, diagnostic logs, Defender alerts)?
- [ ] Data residency requirements satisfied?

## Severity Classification

| Severity | Examples | Action |
|---|---|---|
| **Critical** | Public data exposure, no authentication, plaintext secrets in code, public storage with anonymous access | **VETO. Block deployment.** Fix immediately before proceeding. |
| **High** | Over-permissioned identity (Owner at subscription), missing encryption on sensitive data, no MFA | **Block production.** Must fix within 5 business days. |
| **Medium** | Missing diagnostic settings, non-optimal NSG rules, default TLS without explicit enforcement | **Track in backlog.** Fix within 30 days. |
| **Low** | Best practice deviation, hardening opportunity, cosmetic security improvements | **Informational.** Fix within 90 days or accept risk with documentation. |

## Review Output

Every security review produces a findings document:

| Field | Description |
|---|---|
| **Severity** | Critical, High, Medium, or Low |
| **Category** | Identity, Network, Data, Secrets, Monitoring, or Compliance |
| **Finding** | Clear description of the security issue |
| **Risk** | What could go wrong if not addressed |
| **Recommendation** | Specific remediation with Azure configuration details |
| **Effort** | Estimated remediation effort (hours/days) |

## Sign-Off

Review concludes with one of:
- **Approved** — no Critical or High findings. Medium/Low tracked in backlog.
- **Conditionally Approved** — High findings with documented remediation plan and timeline. Security team monitors.
- **Vetoed** — Critical findings present OR unaddressed High findings. Cannot ship. Justification documented.
