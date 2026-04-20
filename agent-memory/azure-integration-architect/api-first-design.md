# API-First Design Patterns

Patterns for designing APIs contract-first before implementation, ensuring consistency across Azure integration services.

## Contract-First Development

### Why API-First
- Define the API contract (OpenAPI/Swagger) before writing any backend code
- Frontend and backend teams work in parallel against the agreed contract
- APIM can import OpenAPI specs directly — mock responses from day one
- Catches design issues early when changes are cheap, not after implementation

### OpenAPI Specification Workflow
1. Design API in OpenAPI 3.0+ (YAML/JSON) — include schemas, examples, error responses
2. Import spec into APIM — auto-generates API definition, operations, and documentation
3. Configure APIM mock responses (`mock-response` policy) for frontend development
4. Implement backend against the same contract — use code generation where possible
5. Replace mock with real backend — frontend never changes

### Contract Design Rules
- Use consistent naming: `camelCase` for JSON properties, `kebab-case` for URL paths
- Every endpoint must document all response codes (200, 400, 401, 403, 404, 500)
- Include `x-ms-examples` for Azure-specific tooling compatibility
- Define reusable schemas in `components/schemas` — avoid inline definitions
- Use `$ref` for shared error response models across operations

## API Design Standards

### Resource Naming
- Use nouns for resources: `/orders`, `/customers/{id}/addresses`
- Use verbs only for actions that don't map to CRUD: `/orders/{id}/cancel`
- Plural nouns for collections, singular for sub-resources: `/orders/{id}`
- Maximum 3 levels of nesting — deeper hierarchies indicate poor resource modeling

### Pagination
- Use cursor-based pagination for large datasets: `?continuationToken=abc`
- Alternatively, offset-based: `?skip=20&top=10` (consistent with OData and Azure conventions)
- Always include `nextLink` in response for next page — clients should never construct pagination URLs
- Return total count only when explicitly requested (`?$count=true`) — counting is expensive

### Error Responses
- Use RFC 7807 Problem Details format for all errors
- Include `code` (machine-readable), `message` (human-readable), `target` (field), `details` (nested errors)
- Map HTTP status codes consistently: 400 = validation, 401 = unauthenticated, 403 = unauthorized, 404 = not found, 409 = conflict, 429 = throttled
- Never expose stack traces or internal details in production error responses

### Versioning in Practice
- URL path versioning (`/v1/orders`) is the default recommendation — most discoverable, most cacheable
- APIM version sets manage lifecycle: create version → mark preview → GA → deprecated → sunset
- Breaking changes require a new version: removing fields, changing types, altering required fields
- Non-breaking changes go in the current version: adding optional fields, new endpoints, new optional parameters

## APIM as API Design Platform

### Design-Time Features
- **Import OpenAPI**: Upload spec, APIM generates operations, validation schemas, and developer portal docs
- **Schema validation policy**: `validate-content` policy validates request/response bodies against OpenAPI schemas at runtime
- **Named values**: Externalize environment-specific values (URLs, keys) — reference in policies with `{{named-value}}`
- **Backends**: Define backend services with circuit breaker, load balancing — decouple API definition from implementation

### Developer Portal
- Auto-generated from API definitions — interactive "Try It" console for every operation
- Customize with company branding, custom pages, and widgets
- Product grouping: bundle APIs into products with different rate limits, access policies, and subscription tiers
- Self-service subscription: developers request access, admins approve — reduces onboarding friction

## API Governance

### Linting and Validation
- Use Spectral or Azure API Center to lint OpenAPI specs before APIM import
- Enforce rules: no missing descriptions, no inline schemas, consistent error formats, required auth definitions
- Run linting in CI/CD pipeline — fail the build on violations

### API Catalog
- Azure API Center: centralized inventory of all APIs across the organization
- Register APIs regardless of where they're hosted (APIM, App Service, Functions, external)
- Track API lifecycle stages: design → development → testing → production → deprecated → retired
- Link APIs to owners, documentation, environments, and deployments

### Consistency Across Teams
- Publish an API design guide (naming, pagination, error format, versioning) — enforce via linting rules
- Use APIM policy fragments for shared policy logic — avoid copy-paste across APIs
- Review API designs before implementation — treat the OpenAPI spec as a code review artifact
- Template starter APIs in APIM — new teams clone and customize rather than designing from scratch
