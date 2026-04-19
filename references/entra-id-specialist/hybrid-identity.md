## Hybrid Identity — Sync Methods

### Comparison Table

| Feature | Password Hash Sync (PHS) | Pass-Through Auth (PTA) | Federation (AD FS) | Cloud Sync |
|---------|------------------------|------------------------|-------------------|------------|
| **Where auth happens** | Cloud (Microsoft validates hash) | Cloud + on-prem agent validates password | On-premises (AD FS) | Cloud (same as PHS) |
| **On-prem infrastructure** | Entra Connect only | Entra Connect + PTA agents | AD FS farm + WAP servers | Lightweight provisioning agent |
| **Complexity** | Low | Medium | High | Low |
| **Resilience** | High (cloud-based) | Depends on agent availability | Depends on AD FS farm | High (cloud-based) |
| **Password policies** | Entra ID policies | On-prem AD policies enforced at auth | On-prem AD policies | Entra ID policies |
| **Smart lockout** | Yes (cloud-based) | Yes (on-prem lockout) | Depends on AD FS config | Yes (cloud-based) |
| **Leaked credential detection** | Yes (Entra ID Protection) | No | No | Yes (Entra ID Protection) |
| **Multi-forest** | Yes | Yes (agent per forest) | Yes (complex) | Yes (designed for this) |
| **Microsoft recommendation** | **Recommended for most orgs** | Use when passwords MUST stay on-prem | Legacy, migrate away | **Strategic direction** |

### Cloud Sync vs Entra Connect
| Aspect | Entra Connect | Cloud Sync |
|--------|--------------|------------|
| **Architecture** | Heavy on-prem server | Lightweight agent |
| **Agent updates** | Manual or auto-update | Auto-updated from cloud |
| **Multi-forest** | One server per forest typically | Multiple agents, any forest |
| **Sync interval** | 30 minutes default | 2 minutes |
| **Configuration** | On-prem wizard, complex | Cloud-based portal, simpler |
| **Feature parity** | Full (device writeback, group writeback, etc.) | Growing (check current support) |
| **Microsoft direction** | Maintenance mode | **Strategic investment** |

### Migration Path: AD FS → Cloud Auth
```
Step 1: Enable PHS alongside AD FS (as fallback)
Step 2: Test staged rollout — move users to PHS in batches
Step 3: Convert domains from federated → managed
Step 4: Decommission AD FS
Step 5: Clean up — remove trusts, certificates, WAP servers

Key risks:
- Custom claims rules in AD FS → need equivalent in Entra ID token config
- Third-party MFA integrated with AD FS → move to Entra MFA or external auth methods
- On-prem apps using AD FS → migrate to Application Proxy or SAML/OIDC in Entra
```
