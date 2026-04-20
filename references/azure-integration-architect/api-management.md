# API Management

## Architecture Components

- **Gateway**: Processes API requests — routing, policy enforcement, authentication, caching, transformation.
- **Management plane**: Portal/API for configuring APIs, products, policies, users, analytics.
- **Developer portal**: Auto-generated, customizable portal for API consumers to discover, test, and subscribe.

## Tiers

| Tier | Use Case | VNet Support | Multi-Region | AZ | SLA |
|---|---|---|---|---|---|
| **Consumption** | Serverless, variable traffic, pay-per-call | No | No | No | Yes |
| **Developer** | Dev/test, evaluation | Injection | No | No | No SLA |
| **Basic** | Entry production | Private endpoint | No | No | Yes |
| **Basic v2** | Dev/test with SLA, fast provisioning | No | No | No | Yes |
| **Standard** | Mid production | Private endpoint | No | No | Yes |
| **Standard v2** | Production with VNet integration | VNet integration + PE | No | No | Yes |
| **Premium** | Enterprise, full isolation | Full injection | Yes | Yes | Yes |
| **Premium v2** | Enterprise (new generation) | Full injection + integration | No* | Yes | Yes |

*Premium v2 does not yet support multi-region deployments.

## Tier Selection

- **Consumption**: Microservices, serverless, pay-per-call. No developer portal by default. Cold start latency (~1-2s).
- **Standard v2**: Most production workloads. VNet integration for backend access. Fast provisioning (~5 min vs hours).
- **Premium**: Multi-region, full VNet injection, availability zones, self-hosted gateway support. ~$2,794/unit/month.
- **v2 tiers**: Simplified, faster provisioning, better price-performance. Prefer v2 for new deployments.

## Policies (Core Power of APIM)

XML-based pipeline: `inbound` → `backend` → `outbound` → `on-error`

### Policy Scoping
Global → Product → API → Operation. Use `<base />` to inherit parent policies. Most specific scope wins for conflicts.

### Policy Categories

**Authentication (Inbound)**:
- `validate-jwt`: Validate OAuth 2.0 JWT tokens. Check issuer, audience, claims.
- `validate-azure-ad-token`: Simplified Azure AD token validation.
- `check-header`: Verify required headers exist.
- Client certificate validation for mutual TLS.

**Rate Limiting (Inbound)**:
- `rate-limit`: Fixed rate limit per subscription.
- `rate-limit-by-key`: Flexible rate limiting by expression (IP, user, custom key).
- `quota` / `quota-by-key`: Usage quotas over time periods.

**Transformation**:
- `set-header`, `set-body`, `rewrite-uri`: Modify requests/responses.
- `json-to-xml`, `xml-to-json`: Format conversion.
- `find-and-replace`, `xslt-transform`: Content manipulation.

**Caching**:
- `cache-lookup` / `cache-store`: Response caching.
- `cache-lookup-value` / `cache-store-value`: Fragment caching for partial data.

**Backend**:
- `set-backend-service`: Route to different backends conditionally.
- `forward-request`: Control backend call behavior.
- `send-request` / `send-one-way-request`: Make additional HTTP calls from policies.

**Cross-Cutting**:
- `cors`: CORS configuration.
- `ip-filter`: Allow/deny IP ranges.
- `log-to-eventhub`: Stream request data to Event Hubs.
- `emit-metric`: Custom metrics for monitoring.
- `retry`: Retry failed backend calls with configurable policy.
- `choose`: Conditional policy execution (if/else).
- `mock-response`: Return mock responses for testing.

### Policy Expressions
C# inline expressions for dynamic behavior: `@(context.Request.Headers["X-Custom"])`. Access request, response, context variables.

## Products and Subscriptions

- **Products**: Bundle APIs with access policies, rate limits, terms of use.
- **Protected products**: Require subscription key. Default and recommended.
- **Open products**: No subscription required — use OAuth, certs, or IP filtering instead.
- **Subscription scopes**: All APIs, single product, or single API.
- **Subscription keys**: `Ocp-Apim-Subscription-Key` header (default) or `subscription-key` query param.
- **Best practice**: Strip subscription key from backend forwarding — don't leak keys to backends.

## Versioning and Revisions

- **Versions** (breaking changes): URL path (`/v1/`), query string (`?api-version=`), or header (`Api-Version`).
- **Revisions** (non-breaking changes): Test changes in a revision, then make it current. No consumer impact during development.
- Pattern: Use revisions for iterative development, versions only for breaking contract changes.

## Named Values
- Key-value configuration store. Reference in policies with `{{named-value-name}}`.
- Can point to Azure Key Vault secrets for secure values.
- Use for connection strings, API keys, feature flags referenced in policies.

## Developer Portal
- Auto-generated API documentation with interactive test console.
- Subscription self-service management.
- Can self-host for full UI customization.
- **Delegation**: Integrate sign-up/sign-in and subscription workflows with external identity/billing providers.

## Backend Configuration
- Configure backend entities for reuse across APIs.
- Set backend URL, credentials, load balancing across multiple backends.
- **Circuit breaker** (preview): Trip on 5xx count/percentage with auto-recovery. Prevents cascade failures.

## Self-Hosted Gateway
- Run APIM gateway on your own infrastructure (Kubernetes, Docker, Arc).
- Available in Developer and Premium tiers.
- Use for: low-latency on-prem backend access, compliance requiring local processing, edge deployments.
- Syncs configuration from APIM management plane. Operates independently if connection lost.
