# Entra ID Gotchas

Non-obvious behaviors and subtle distinctions that frequently trip up engineers working with Entra ID.

---

## App Registration vs Enterprise App (Service Principal)

- An **app registration** is the application definition (template) — lives in your home tenant
- An **enterprise application** is the **service principal** — the local instance in a tenant that actually gets permissions and sign-in logs
- Deleting the app registration deletes service principals in all tenants; deleting the enterprise app only removes the local instance
- You can have an enterprise app without an app registration (e.g., gallery apps provisioned from Entra)
- Permissions configured on the app registration are *requested*; permissions on the service principal are *granted*

## Conditional Access Policies Are AND'd, Not OR'd

- When multiple CA policies apply to a sign-in, **all** applicable policies must be satisfied
- If Policy A requires MFA and Policy B requires compliant device, the user needs **both**
- There is no "first match wins" — every matching policy's grant controls are combined
- A single "Block" policy overrides all "Grant" policies
- Use the **What If** tool to simulate which policies apply to a specific sign-in scenario

## PIM Role Activation Propagation Delay

- After activating a PIM role, the assignment can take **up to 5 minutes** to propagate
- Token caches on the client may take longer — user may need to sign out and back in
- Azure portal may show the role as active while API calls still return 403
- **Workaround:** Force a fresh token by clearing browser session or calling `/me` on Graph to verify role presence

## Workload Identity Federation: Token Audience Must Match Exactly

- The `audience` field in the federated identity credential must exactly match the token's `aud` claim
- For GitHub Actions: `api://AzureADTokenExchange` (default) — do not change unless you customize the OIDC provider
- For AKS: the audience is the cluster's OIDC issuer URL
- Mismatched audience = silent authentication failure with unhelpful error messages
- The `subject` claim is also exact-match — `repo:org/repo:ref:refs/heads/main` won't match `refs/heads/feature`

## Guest User Types: "Member" Guest vs "Guest" Guest

- Entra ID has two axes: **source** (internal vs invited) and **userType** (Member vs Guest)
- You can set `userType=Member` on an invited external user — they get the same default permissions as internal users
- `userType=Guest` gets restricted defaults: can't enumerate directory, can't read other users' full profiles
- Conditional access can target by userType — a "Member" guest may bypass policies targeting "Guest" users
- Check both `creationType` and `userType` when auditing external identities

## B2C vs External ID: Different Products

- **Azure AD B2C** is a separate, standalone service with its own tenant type — uses custom policies (XML) or user flows
- **Microsoft Entra External ID** (formerly "External Identities") is native to Entra ID — uses external tenants
- B2C tenants cannot be merged into regular Entra tenants
- External ID supports OIDC/SAML natively; B2C requires custom policy for SAML
- New greenfield customer identity projects should evaluate External ID first; B2C is legacy-path

## Graph API: Application vs Delegated Permissions

- **Delegated** permissions act on behalf of a signed-in user — scoped to what the user can access
- **Application** permissions act as the service itself — no user context, typically broader access
- Application permissions require **admin consent**; some delegated permissions can be user-consented
- `User.Read` delegated = read the signed-in user. `User.Read.All` application = read every user in the directory
- Application permissions ignore user-level conditional access policies

## Managed Identity: System-Assigned Lifecycle

- **System-assigned** managed identity is created and deleted with the Azure resource (VM, App Service, etc.)
- If you delete and recreate the resource, you get a **new** identity with a new object ID — all role assignments are lost
- **User-assigned** managed identity survives resource deletion and can be shared across resources
- System-assigned is simpler but creates tight coupling; user-assigned is better for blue-green deployments
- Managed identities can't be used cross-tenant (they exist only in the resource's home tenant)

## Token Lifetime: Inactivity Timeout

- Refresh tokens have **both** an absolute lifetime and an **inactivity timeout** (default: 90 days inactive)
- If a refresh token isn't used for 90 days, it expires even if the absolute lifetime hasn't been reached
- Continuous Access Evaluation (CAE) can revoke tokens near-instantly for critical events
- Access tokens default to ~60-75 minutes; customizing via token lifetime policies is limited post-2021
- Configurable token lifetime policies only apply to access, ID, and SAML tokens — not refresh tokens (use CAE instead)

## Conditional Access: Named Locations Use IP Ranges

- Named locations are defined by **IP address ranges** (IPv4/IPv6 CIDR) or countries (GeoIP)
- There is no DNS-based named location — you cannot use hostnames or FQDNs
- Country-based locations rely on IP geolocation databases — VPNs and cloud egress IPs may resolve to unexpected countries
- GPS-based named locations require the Authenticator app and are only supported on mobile devices
- Overlapping IP ranges across named locations can cause unpredictable policy evaluation

## Service Principal Sign-In Logs Are Separate

- **User sign-ins**, **service principal sign-ins**, and **managed identity sign-ins** are three separate log categories
- Most teams only monitor user sign-ins — service principal compromise goes undetected
- Service principal sign-in logs require at least Entra ID P1 license
- In Log Analytics, service principal sign-ins are in `AADServicePrincipalSignInLogs`, not `SigninLogs`
- Diagnostic settings must explicitly include all three categories when exporting logs

---

> **Sources:** [Application and service principal objects](https://learn.microsoft.com/en-us/entra/identity-platform/app-objects-and-service-principals) · [Conditional access policy evaluation](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-policies) · [Workload identity federation](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation) · [Token lifetimes](https://learn.microsoft.com/en-us/entra/identity-platform/configurable-token-lifetimes)
