## B2B and B2C / External ID

### B2B (Business-to-Business)

Guest users from other organizations accessing your resources:

| Aspect | Details |
|--------|---------|
| **Purpose** | Partners, vendors, contractors accessing your tenant's resources |
| **Identity source** | Their own Entra ID tenant, Google, SAML/WS-Fed IdP, or email OTP |
| **User type** | Guest in your directory (UserType = Guest) |
| **Licensing** | Your tenant needs P1/P2 for CA policies on guests |
| **Governance** | Access reviews, entitlement management, access packages |

### Cross-Tenant Access Settings

- **Inbound**: control how external users access your resources
- **Outbound**: control how your users access external resources
- **Default settings**: apply to all tenants not explicitly configured
- **Per-tenant overrides**: specific trust settings for key partners
- **Trust settings**: trust MFA, compliant device, hybrid joined device from partner tenant

### External Collaboration Policies

- Who can invite guests: admins only, members, or guests too
- Guest access level: same as members, limited, or most restricted
- Collaboration restrictions: allow/block specific domains
- Guest self-service sign-up via user flows
- Review and clean up stale guest accounts regularly

### B2C / External ID (Customer-Facing Identity)

| Aspect | Details |
|--------|---------|
| **Purpose** | Customer-facing applications with self-service sign-up |
| **Identity source** | Local accounts, Google, Facebook, Apple, SAML/OIDC providers |
| **Scale** | Millions of users, consumer-grade UX |
| **Customization** | Custom sign-up/sign-in flows, branding, attribute collection |
| **Directory** | Separate B2C tenant or External ID in workforce tenant |

### Social Identity Providers

- Google, Facebook, Apple, Microsoft account, Twitter/X
- Configure via identity provider settings in B2C tenant
- Map claims from social IdP to B2C user attributes
- Support for custom OIDC and SAML providers

### Custom Policies

- Identity Experience Framework (IEF) for complex flows
- XML-based policy definitions for advanced scenarios:
  - Multi-step verification, progressive profiling
  - API calls during sign-up for validation
  - Custom claim transformations
  - Conditional logic based on user attributes
- User flows (portal-based) sufficient for most standard scenarios

### Decision: B2B vs B2C

```
External user accessing your corporate resources?
  → B2B (guest user with their own org identity)

Customer-facing app with self-service registration?
  → B2C or External ID

Need social login (Google, Facebook, Apple)?
  → B2C / External ID

Modern stack, simpler requirements?
  → External ID (Microsoft's strategic direction)

Complex custom auth flows?
  → B2C custom policies (Identity Experience Framework)
```
