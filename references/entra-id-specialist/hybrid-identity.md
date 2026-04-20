## Hybrid Identity

### Entra Connect vs Cloud Sync

| Aspect | Entra Connect | Cloud Sync |
|--------|--------------|------------|
| **Architecture** | On-prem server (heavy) | Lightweight cloud agent |
| **Agent updates** | Manual or scheduled | Auto-updated from cloud |
| **Multi-forest** | One server per forest typically | Multiple agents, any forest |
| **Sync interval** | 30 minutes default | 2 minutes |
| **Configuration** | On-prem wizard | Cloud portal |
| **Feature parity** | Full (device writeback, group writeback) | Growing — check current support |
| **Microsoft direction** | Maintenance mode | Strategic investment |

### Authentication Methods

| Method | Where Auth Happens | Complexity | Recommendation |
|--------|--------------------|------------|----------------|
| **Password Hash Sync (PHS)** | Cloud — Microsoft validates hash | Low | Recommended for most orgs |
| **Pass-Through Auth (PTA)** | Cloud + on-prem agent validates | Medium | When passwords MUST stay on-prem |
| **Federation (AD FS)** | On-premises AD FS farm | High | Legacy — migrate away |

### Why PHS Is Recommended

- Simplest to deploy and maintain — no on-prem infrastructure dependency
- Enables **leaked credential detection** via Entra ID Protection
- Smart lockout works in the cloud
- High availability — not dependent on on-prem agents
- Works with seamless SSO for on-prem joined devices
- Microsoft's recommended default for most organizations

### PTA — When to Use

- Regulatory requirement that password hashes cannot leave on-premises
- Enforces on-prem AD password policies at sign-in time
- Requires PTA agents on-prem (at least 3 for high availability)
- Falls back to nothing if all agents are offline — plan accordingly

### Federation (AD FS) — When to Migrate Away

- High operational cost: AD FS farm + WAP servers + certificates
- On-prem dependency for every authentication
- Custom claims rules add complexity
- Microsoft actively recommends migration to PHS

### Seamless SSO

- Automatic sign-in for domain-joined devices on corporate network
- Works with PHS and PTA (not needed with federation)
- Uses Kerberos — device gets ticket, Entra ID validates
- No user interaction required on managed devices

### Device Registration

| State | Description | Management |
|-------|-------------|------------|
| **Entra joined** | Cloud-only device, joined to Entra ID | Intune MDM |
| **Hybrid Entra joined** | Domain-joined + registered in Entra ID | GPO + Intune co-management |
| **Entra registered** | Personal device, registered for SSO | MAM (app-level) |

### Migration Path: AD FS → PHS

```
Step 1: Enable PHS alongside AD FS (as backup/fallback)
Step 2: Staged rollout — move user groups to PHS in batches
Step 3: Convert domains from federated to managed
Step 4: Decommission AD FS farm
Step 5: Clean up — remove trusts, certificates, WAP servers

Key migration risks:
- Custom claims rules → map to Entra ID token configuration
- Third-party MFA on AD FS → migrate to Entra MFA
- On-prem apps using AD FS → Application Proxy or SAML/OIDC in Entra
- Test thoroughly in staged rollout before full cutover
```
