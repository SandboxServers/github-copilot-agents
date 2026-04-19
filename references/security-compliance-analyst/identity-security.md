## Identity Security Review

### Least Privilege Checklist
- **Entra ID roles**: No permanent Global Admin assignments. Use PIM for just-in-time activation.
- **Custom roles**: Prefer custom roles scoped to specific operations over broad built-in roles.
- **Service principals**: Scoped to specific resource groups, not subscriptions. Time-limited credentials.
- **Managed identity**: Prefer system-assigned for single-resource scenarios, user-assigned for shared identity.
- **Conditional access**: Require MFA for all admin roles. Block legacy authentication. Require compliant devices.
- **Break-glass accounts**: Minimum 2 cloud-only accounts, excluded from conditional access, monitored with alerts, credentials stored in physical safe.

### Service Principal Security
1. **Scope minimally** — assign RBAC at resource group level, not subscription
2. **Use certificates** over client secrets where possible
3. **Rotate credentials** — set expiration, automate rotation via Key Vault
4. **Audit regularly** — identify stale/unused service principals via Entra ID sign-in logs
5. **Prefer workload identity federation** — eliminate secrets entirely for GitHub Actions, AKS, etc.
6. **No owner permissions** — service principals should never have Owner role

### Managed Identity Best Practices
- System-assigned for resources that need their own identity (VMs, App Services, Functions)
- User-assigned when multiple resources share the same identity or when identity lifecycle differs from resource
- Grant only the specific RBAC roles needed (e.g., Storage Blob Data Reader, not Contributor)
- Use managed identity for Key Vault access, database connections, storage access — eliminate connection strings
