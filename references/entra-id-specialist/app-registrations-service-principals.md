## App Registrations vs Enterprise Apps (Service Principals)

### The Relationship
```
App Registration (Application Object)
├── Lives in your home tenant
├── Globally unique — one per app
├── YOU control: redirect URIs, credentials, API permissions, branding
├── Created in: Azure Portal → App registrations, or Microsoft Graph
│
└── Creates → Service Principal (Enterprise Application)
    ├── Instance of the app in EACH tenant where users authenticate
    ├── Tenant admin controls: user assignment, conditional access, SSO config
    ├── For multi-tenant apps: SP exists in every consenting tenant
    └── Created automatically (OIDC/OAuth) or admin-initiated (SAML)
```

### Key Differences

| Aspect | App Registration | Enterprise App (Service Principal) |
|--------|-----------------|-----------------------------------|
| **What it is** | Global app definition (the blueprint) | Instance in a specific tenant (the deployment) |
| **Who controls it** | App developer/ISV | Tenant admin |
| **Where it lives** | Home tenant only | Every tenant that uses the app |
| **What you configure** | Redirect URIs, credentials, API permissions, token configuration | User assignment, SSO settings, conditional access, provisioning |
| **OIDC/OAuth apps** | Created first → SP created automatically | Auto-created on first consent |
| **SAML apps** | Created automatically by Entra | Created first (admin adds from gallery or custom) |
| **PowerShell** | `Get-MgApplication` | `Get-MgServicePrincipal` |

### Common Confusion Points
- **OIDC/OAuth flow**: Register app first → SP auto-created → configure SSO on SP
- **SAML flow**: Create Enterprise App first → App registration auto-created → configure SAML on SP
- **Gallery apps**: Pre-configured templates, one-click setup, Microsoft-maintained integrations
- **Non-gallery apps**: Custom SAML or OIDC apps, manual configuration required
- **Multi-tenant apps**: One app registration, one SP per consuming tenant
