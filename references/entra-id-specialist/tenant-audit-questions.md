## Tenant Audit Checklist

### Service Principal Hygiene
```
Find over-permissioned service principals:
1. List all app registrations with Application-level (not delegated) permissions
2. Flag any with: Directory.ReadWrite.All, RoleManagement.ReadWrite.Directory, 
   Application.ReadWrite.All, or *.ReadWrite.All
3. Check credential expiry — find SPs with credentials expiring > 2 years
4. Find SPs with no recent sign-in activity (stale)
5. Find SPs created by users who have left the organization
6. Check for SPs with owner = departing employee

PowerShell/Graph queries:
- Get-MgServicePrincipal | Where-Object {$_.AppRoleAssignments}
- Check signInActivity on servicePrincipal via Graph beta API
- Review App registrations → Certificates & secrets → expiry dates
```

### Identity Security Posture
- [ ] Security defaults OR conditional access (not both — CA replaces defaults)
- [ ] Legacy auth blocked via conditional access
- [ ] MFA required for all users
- [ ] Phishing-resistant MFA for admins (FIDO2, Windows Hello, certificate-based)
- [ ] Break-glass accounts created, excluded from CA, monitored
- [ ] PIM enabled for all admin roles
- [ ] Access reviews configured for privileged roles
- [ ] Workload identity federation replacing secrets where possible
- [ ] No service principals with unnecessary Global Admin
- [ ] Guest access scoped and reviewed
- [ ] Entra ID Protection policies enabled (user risk + sign-in risk)

## Common Mistakes & Anti-Patterns

1. **Permanent Global Admin assignments** → use PIM eligible assignments for everyone except break-glass
2. **No break-glass accounts** → conditional access change locks out all admins → recovery requires Microsoft support
3. **Break-glass accounts IN conditional access policies** → defeats the entire purpose
4. **Blocking legacy auth without checking sign-in logs** → breaks printers, scanners, legacy apps
5. **Client secrets on app registrations with no expiry tracking** → expired secret = outage at 2am
6. **Service principals with Directory.ReadWrite.All** → can modify any object in the directory
7. **No access reviews for guests** → stale guest accounts with persistent access
8. **Single conditional access policy trying to do everything** → impossible to troubleshoot, maintain, or exclude
9. **ROPC flow in production** → bypasses MFA, conditional access, breaks on any policy change
10. **Federation with AD FS when PHS would work** → unnecessary complexity, on-prem dependency, harder to secure

## Questions This Specialist Always Asks

### Authentication Architecture
- What auth method? (PHS, PTA, federation, cloud-only?) — if federation, why not PHS?
- Any AD FS dependencies? What custom claims rules exist?
- What's the MFA story? (Entra MFA, external provider, NPS extension?)
- Legacy auth — is it blocked? Have you checked sign-in logs for usage?

### Application Landscape
- How many app registrations in the tenant? When was the last audit?
- Any service principals with application-level permissions to Graph?
- Multi-tenant apps exposing APIs? How is consent managed?
- Workload identity federation in use for CI/CD? Or still using secrets?

### Privileged Access
- How many permanent Global Admins? (Answer should be 2 — break-glass only)
- Is PIM enabled? What roles are covered? What's the activation process?
- Access reviews configured? When was the last one completed?
- Break-glass accounts — do they exist, are they excluded from CA, are they monitored?

### External Identity
- B2B guests — how many, how old, last sign-in activity?
- Guest access policies — can guests enumerate the directory?
- B2C/External ID — customer-facing apps using which identity platform?

### Hybrid Identity
- Entra Connect or Cloud Sync? Version? Health status?
- Sync scope — all users or filtered? OU filtering or group filtering?
- Password writeback enabled? Self-service password reset configured?
- Plans to migrate from AD FS to cloud auth?
