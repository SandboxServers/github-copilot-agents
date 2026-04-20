## API Testing

### Contract Testing

- Validate APIs against their OpenAPI (Swagger) specification on every build
- Use schema validators to ensure responses match documented structure
- **Pact** — consumer-driven contract testing between services
  - Consumer defines expected interactions
  - Provider verifies it can fulfill those contracts
  - Contracts are versioned and shared via a Pact Broker
- Catch breaking changes before deployment — field removals, type changes, required fields
- Contract tests run faster than full integration tests and provide precise failure signals

### Integration Testing for APIs

- Test API endpoints with real (or containerized) dependencies
- Validate the full request-response cycle: serialization, middleware, business logic, persistence
- Use test containers (Testcontainers) for databases, caches, and message brokers
- Test with realistic payloads — not just happy-path minimal examples
- Validate timeout handling, retry behavior, and circuit breaker activation

### Status Code Validation

| Scenario | Expected Code | What to Verify |
|----------|--------------|----------------|
| Successful create | 201 Created | Location header, resource in response body |
| Successful update | 200 OK | Updated fields reflected in response |
| Resource not found | 404 Not Found | Consistent error body format |
| Validation failure | 400 Bad Request | Field-level error details in response |
| Authentication missing | 401 Unauthorized | No sensitive data leaked in error |
| Authorization denied | 403 Forbidden | Appropriate error message |
| Rate limit exceeded | 429 Too Many Requests | Retry-After header present |
| Server error | 500 Internal Server Error | No stack traces or internal details exposed |

### Response Schema Validation

- Validate every response against the OpenAPI schema automatically
- Check required fields are always present — never null when non-nullable
- Validate data types, formats (date-time, email, UUID), and enum values
- Verify pagination metadata (total count, page size, next link) is consistent
- Test with various Accept headers to ensure content negotiation works

### Authentication Testing

- Verify endpoints reject requests without valid credentials (401)
- Test token expiration handling — expired tokens return 401, not 500
- Validate scope/role-based access — users without permission get 403
- Test API key rotation — old keys rejected after rotation
- Verify OAuth flows: authorization code, client credentials, refresh tokens
- Ensure authentication errors do not leak internal details

### Rate Limiting Testing

- Verify rate limit headers are returned (X-RateLimit-Limit, X-RateLimit-Remaining)
- Confirm 429 response when limit is exceeded
- Validate Retry-After header provides accurate wait time
- Test rate limit reset behavior — limits should reset after the window
- Test rate limiting per client/API key, not globally

### Error Response Format Consistency

- All error responses should follow a consistent schema across the entire API
- Include: error code, human-readable message, request ID for correlation
- Never expose stack traces, internal paths, or database details in errors
- Validate error responses for every error status code (400, 401, 403, 404, 500)
- Use a shared error model defined in the OpenAPI specification

### Backward Compatibility Testing

- Run previous API version's test suite against the new version
- Verify existing clients continue to work without modification
- Test that deprecated fields are still returned (with deprecation warnings if applicable)
- Validate that new required fields have sensible defaults for existing clients
- Use API versioning (URL path, header, or query parameter) and test each supported version
- Document and communicate breaking changes with migration guides
