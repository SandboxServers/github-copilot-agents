## App Registrations and Service Principals

### App Registration = Identity Configuration

An app registration defines the app's identity in your home tenant:
- **Redirect URIs**: where tokens are sent after authentication
- **Credentials**: certificates (preferred) or client secrets
- **API permissions**: what the app is allowed to access
- **Token configuration**: optional claims, token version, group claims
- **Expose an API**: define scopes for other apps to consume
- **Branding**: publisher domain, logo, terms of service

### Enterprise App (Service Principal) = Instance in Tenant

- App registration creates the blueprint; service principal is the deployed instance
- For multi-tenant apps: one app registration, one service principal per consuming tenant
- Tenant admin controls: user assignment, conditional access, SSO config, provisioning
- OIDC/OAuth: register app first → SP auto-created on first consent
- SAML: create enterprise app first → app registration auto-created

### Multi-Tenant vs Single-Tenant

| Aspect | Single-Tenant | Multi-Tenant |
|--------|--------------|--------------|
| **Audience** | Your org only | Any Entra ID tenant |
| **Supported accounts** | This org directory only | Accounts in any org directory |
| **Service principals** | One (your tenant) | One per consenting tenant |
| **Use case** | Internal LOB apps | SaaS apps, ISV products |
| **Consent** | Admin consent in your tenant | Each tenant admin consents |

### API Permissions

**Delegated permissions** (user context):
- App acts on behalf of the signed-in user
- Effective permissions = intersection of app permissions AND user's own permissions
- Example: `User.Read` — app can read the signed-in user's profile

**Application permissions** (daemon/service context):
- App acts as itself with no signed-in user
- Effective permissions = exactly what's granted (no user intersection)
- Example: `Mail.Read` — app can read ALL users' mail
- Always requires admin consent

### Permission Types

| Source | Examples | Notes |
|--------|----------|-------|
| **Microsoft Graph** | User.Read, Mail.Send, Directory.Read.All | Most common API surface |
| **Custom APIs** | api://your-app-id/access_as_user | Your own APIs exposed via app registration |
| **Azure Service Management** | user_impersonation | Azure Resource Manager access |
| **Other Microsoft APIs** | SharePoint, Exchange, Dynamics | Service-specific permissions |

### Consent Framework

**User consent**: user grants delegated permissions for themselves
- Controlled by user consent settings in Entra ID
- Can be restricted to pre-approved permissions or disabled entirely

**Admin consent**: tenant admin grants permissions for all users
- Required for application permissions (always)
- Required for high-privilege delegated permissions
- Admin consent workflow: users request, admins approve

### Best Practices

- **Least privilege**: request only permissions the app actually needs
- **Certificates over secrets**: certificates are more secure, can't be accidentally logged
- **Short-lived secrets**: if secrets are necessary, set 6-month or shorter expiry
- **Track expiry**: monitor credential expiry, alert 30 days before
- **Review regularly**: audit unused app registrations and stale permissions
- **Named owners**: every app registration should have at least 2 identifiable owners
- **No wildcard permissions**: avoid `*.ReadWrite.All` unless absolutely necessary
