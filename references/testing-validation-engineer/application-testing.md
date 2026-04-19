## Application Testing

### Test Types for Azure Workloads
| Type | What It Validates | Tools |
|---|---|---|
| **Unit tests** | Business logic works as expected | NUnit, xUnit, Jest, pytest |
| **Integration tests** | Components work together | xUnit + real dependencies, Testcontainers |
| **Smoke tests** | Infrastructure + app available and responding | Playwright, curl, custom health checks |
| **UI/E2E tests** | User flows work end-to-end | Playwright (recommended), Selenium, Cypress |
| **Load tests** | Scalability, performance under load | Azure Load Testing (JMeter/Locust), k6 |
| **Stress tests** | Breaking point, graceful degradation | Azure Load Testing with escalating load |
| **Chaos tests** | Resiliency under failure conditions | Azure Chaos Studio |
| **Penetration tests** | Security posture, vulnerability detection | OWASP ZAP, Burp Suite, Microsoft security tools |

### Testing Theater vs Useful Testing
**Testing theater** (avoid):
- 100% code coverage on getters/setters/DTOs
- Tests that only verify mocking framework works
- UI tests for every page that break on any CSS change
- Tests that pass even when the underlying behavior is broken

**Useful testing** (pursue):
- Tests that validate business rules and edge cases
- Tests that catch regressions before customers do
- Tests that verify integration points (auth, data, external APIs)
- Tests that validate error handling and failure modes
