## Authentication Flows

### OAuth 2.0 Flows

| Flow | Use Case | Client Type | Key Characteristics |
|------|----------|-------------|---------------------|
| **Authorization Code** | Server-side web apps | Confidential | Back-channel token exchange, server holds secret |
| **Authorization Code + PKCE** | SPAs, mobile apps, desktop apps | Public | Code verifier/challenge prevents interception, no client secret needed |
| **Client Credentials** | Service-to-service, daemons, background jobs | Confidential | No user context, app authenticates with its own identity |
| **On-Behalf-Of (OBO)** | Middle-tier API calling downstream API | Confidential | Preserves delegated user context through service chain |
| **Device Code** | CLI tools, IoT devices, smart TVs | Public | User authenticates on a separate device with a browser |

### When to Use Each Flow

```
Web app with server backend?
  → Authorization Code (confidential client with secret/cert)

Single-page app (React, Angular)?
  → Authorization Code + PKCE (public client, no secret)

Mobile or desktop app?
  → Authorization Code + PKCE (public client)

Service-to-service with no user?
  → Client Credentials (app identity, app roles)

API calling another API on behalf of user?
  → On-Behalf-Of (chained delegation)

CLI tool or device without browser?
  → Device Code (user completes auth elsewhere)

Legacy app with username/password?
  → ROPC — AVOID: no MFA, no conditional access, breaks on policy changes
```

### OpenID Connect (OIDC)

OIDC adds an identity layer on top of OAuth 2.0:
- OAuth 2.0 handles **authorization** (what can you access)
- OIDC handles **authentication** (who are you)
- Returns an **ID token** with user claims alongside the access token
- Discovery document at `/.well-known/openid-configuration`
- Standard scopes: `openid`, `profile`, `email`, `offline_access`

### SAML 2.0

- XML-based federation protocol for enterprise SSO
- Used by legacy and enterprise SaaS applications
- SP-initiated (app redirects to IdP) vs IdP-initiated (portal click)
- Assertion contains authentication statement + attribute statement
- Still common in enterprise but OIDC preferred for new apps

### Token Types

| Token | Purpose | Issued By | Consumed By |
|-------|---------|-----------|-------------|
| **ID Token** | User identity claims | Entra ID | Client application |
| **Access Token** | API authorization (scopes/roles) | Entra ID | Resource API |
| **Refresh Token** | Obtain new access tokens silently | Entra ID | Client application |

### MSAL (Microsoft Authentication Library)

- Use MSAL for all new applications — handles token acquisition, caching, refresh
- Available for: .NET, JavaScript, Python, Java, Go, Angular, React, Node.js
- Handles PKCE, token caching, silent renewal, account management
- Replaces ADAL (deprecated, end of support)
- Confidential client: `ConfidentialClientApplication` (web apps, APIs)
- Public client: `PublicClientApplication` (SPAs, mobile, desktop)

### Deprecated Flows — Do Not Use

| Flow | Why Deprecated | Replacement |
|------|---------------|-------------|
| **Implicit** | Token in URL fragment, no refresh token, XSS risk | Auth Code + PKCE |
| **ROPC** | No MFA, no CA, credentials in app | Interactive flows or Client Credentials |
| **ADAL** | End of support, no new features | MSAL |
