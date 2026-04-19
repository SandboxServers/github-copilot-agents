## Authentication Protocols

### When to Use Which

| Protocol | Use When | Key Characteristics |
|----------|----------|-------------------|
| **OAuth 2.0** | API authorization, delegated access | Authorization only (not authentication), access tokens, scopes |
| **OpenID Connect (OIDC)** | Modern web/mobile/SPA authentication | Built on OAuth 2.0, adds ID tokens, user authentication |
| **SAML 2.0** | Enterprise SSO, legacy apps, SaaS federation | XML-based, assertion tokens, mature enterprise ecosystem |
| **WS-Federation** | Legacy Windows/.NET apps | Being replaced by OIDC, still in some older enterprise apps |

### OAuth 2.0 / OIDC Flows

| Flow | Use Case | Client Type |
|------|----------|-------------|
| **Authorization Code + PKCE** | Web apps, SPAs, mobile apps | Confidential & public clients |
| **Client Credentials** | Service-to-service, daemon apps, no user context | Confidential clients only |
| **On-Behalf-Of (OBO)** | Middle-tier API calling downstream API on behalf of user | Confidential clients |
| **Device Code** | CLI tools, IoT devices, input-constrained devices | Public clients |
| **Implicit** | ~~Legacy SPAs~~ **DEPRECATED — use Auth Code + PKCE instead** | Public clients |
| **ROPC** | ~~Username/password~~ **AVOID — no MFA, no conditional access** | Last resort only |

### Token Types
- **ID Token (OIDC)**: Who the user is — claims about identity, used by the app
- **Access Token (OAuth)**: What the user/app can do — scopes/roles, sent to APIs
- **Refresh Token**: Long-lived token to get new access tokens without re-auth
