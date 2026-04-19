## Policy-Driven Governance

### Policy Deployment Strategy

```
Phase 1: AUDIT ONLY (Week 1-4)
├─ Deploy all policies in Audit/AuditIfNotExists mode
├─ Monitor compliance dashboard
├─ Identify existing non-compliant resources
└─ Communicate findings to teams

Phase 2: REMEDIATE (Week 4-8)
├─ Create remediation tasks for existing resources
├─ Work with teams to fix non-compliance
├─ Grant exemptions where genuinely needed (with expiration dates)
└─ Document all exemptions

Phase 3: ENFORCE (Week 8+)
├─ Switch policies to Deny/DeployIfNotExists
├─ Monitor for new non-compliance
├─ Maintain exemption lifecycle
└─ Iterate: add new policies in Audit first, then promote
```

### Critical Policies (ALZ Default Set)

| Category | Policy Examples | Effect |
|----------|----------------|--------|
| **Network** | No public IP on NICs, NSG required on subnets, no public endpoints on PaaS | Deny |
| **Identity** | No classic administrators, MFA enforcement (via conditional access) | Deny / Audit |
| **Monitoring** | Activity Log to Log Analytics, Defender for Cloud enabled, diagnostic settings | DeployIfNotExists |
| **Tagging** | Required tags (CostCenter, Owner, Environment), tag inheritance | Append / Deny |
| **Compute** | Allowed VM SKUs, allowed regions, no unmanaged disks | Deny |
| **Storage** | HTTPS only, minimum TLS version, no public blob access | Deny / Modify |
| **Governance** | Allowed resource types, region restrictions | Deny |

### Policy Anti-Patterns

1. **Deny everything on day one** → Teams can't deploy anything → backlash → policies get removed entirely
2. **No exemption process** → Teams find workarounds (shadow IT) instead of requesting exemptions
3. **Exemptions without expiration** → "Temporary" exemptions become permanent → governance erodes
4. **Policy at subscription scope** → Should be at management group scope for consistency and inheritance
5. **Custom policies for things built-in policies handle** → Maintenance burden, drift from ALZ defaults
6. **Not testing policies against existing resources** → Deploy Deny policy → existing deployments break on next update
