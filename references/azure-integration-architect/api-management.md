## API Management (APIM)

### Architecture
- **Gateway**: processes API requests — routing, policy enforcement, authentication, caching, transformation
- **Management plane**: portal/API for configuring APIs, products, policies, users
- **Developer portal**: auto-generated portal for API consumers to discover, test, subscribe

### Tier Comparison
| Tier | Use Case | VNet | Multi-region | AZ | SLA |
|---|---|---|---|---|---|
| **Consumption** | Serverless, variable traffic | No | No | No | Yes |
| **Developer** | Dev/test, evaluation | Injection | No | No | No |
| **Basic** | Entry production | Private endpoint | No | No | Yes |
| **Basic v2** | Dev/test with SLA | No | No | No | Yes |
| **Standard** | Mid production | Private endpoint | No | No | Yes |
| **Standard v2** | Production with VNet integration | VNet integration + PE | No | No | Yes |
| **Premium** | Enterprise | Full injection | Yes | Yes | Yes |
| **Premium v2** | Enterprise (new gen) | Full injection + integration | No* | Yes | Yes |

*Premium v2 does not yet support multi-region deployments.

### Key Decision Points
- **Consumption**: microservices, serverless, pay-per-call. No developer portal by default. Cold start latency.
- **Standard v2**: most production workloads. VNet integration for backend access. Fast provisioning.
- **Premium**: multi-region, full VNet injection, availability zones, self-hosted gateway support.
- **Cost gotcha**: Classic Premium is expensive (~$2,800/mo per unit). V2 tiers offer better price-performance for most scenarios.

### Policies (The Real Power)
- XML-based pipeline: `inbound` → `backend` → `outbound` → `on-error`
- **Policy scoping**: Global → Product → API → Operation (use `<base />` to inherit parent policies)
- Key policy categories:
  - **Authentication**: validate-jwt, validate-azure-ad-token, check-header, client certificates
  - **Transformation**: set-header, set-body, rewrite-uri, json-to-xml, xml-to-json, find-and-replace, xslt-transform
  - **Rate limiting**: rate-limit, rate-limit-by-key, quota, quota-by-key
  - **Caching**: cache-lookup, cache-store, cache-lookup-value
  - **Backend**: set-backend-service, forward-request, send-request, send-one-way-request
  - **Cross-cutting**: cors, ip-filter, log-to-eventhub, emit-metric, retry, choose (conditional), mock-response
- **Policy expressions**: C# inline (`@(context.Request.Headers["X-Custom"])`) for dynamic behavior
- **Named values**: key-value store for secrets, connection strings — reference in policies. Can point to Key Vault.

### Products & Subscriptions
- **Products**: group APIs together with access policies, rate limits, terms of use
- **Protected products**: require subscription key for access
- **Open products**: no subscription required (use OAuth, certs, or IP filtering instead)
- **Subscription scopes**: All APIs, single product, single API
- **Subscription keys**: `Ocp-Apim-Subscription-Key` header (default) or `subscription-key` query param
- **Best practice**: remove subscription key from backend forwarding (strip in outbound policy) — don't leak keys to backends

### Developer Portal
- Auto-generated, customizable portal for API consumers
- Displays API documentation, interactive test console, subscription management
- Can self-host for full customization
- **Delegation**: integrate sign-up/sign-in and subscription workflows with external identity/billing providers

### Backends
- Configure backend entities for reuse across APIs
- Set backend URL, credentials, circuit breaker, load balancing
- **Circuit breaker** (preview): configure trip conditions (5xx count/percentage) with auto-recovery
