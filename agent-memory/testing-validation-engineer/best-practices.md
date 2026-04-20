## Testing Best Practices

### Test Pyramid — Distribution

- **70% Unit tests** — fast (<100ms), isolated, mock external dependencies only
- **20% Integration tests** — real dependencies (databases, APIs), run in seconds
- **10% E2E tests** — full system, critical user journeys only, accept brittleness tradeoff
- Shift investment toward the base — every bug found by a unit test is cheaper than one found by E2E
- Revisit ratios as the codebase matures; complex integration surfaces may justify more middle-layer tests

### Infrastructure Testing

- **Terraform:** `terraform validate` → tflint → Checkov → `terraform plan` assertions → `terraform test`
- **Bicep:** bicep linter → `az deployment group validate` → what-if + Pester assertions → smoke tests
- **Policy compliance:** Azure Policy, Checkov, terraform-compliance for pre-deployment guardrails
- Smoke tests post-deployment: verify resource existence, connectivity, and configuration
- Treat infrastructure tests as first-class — they run in CI alongside application tests

### Performance Testing Patterns

| Pattern | Purpose | When to Run |
|---------|---------|-------------|
| **Baseline** | Establish normal performance metrics | After major changes, monthly |
| **Load** | Validate behavior under expected peak | Every release, PR for critical paths |
| **Stress** | Find breaking points beyond expected load | Before major releases |
| **Spike** | Test sudden traffic surges and recovery | Quarterly or before events |
| **Soak/Endurance** | Detect memory leaks, connection exhaustion | Weekly in staging |

- Use Azure Load Testing integrated with CI/CD for automated regression detection
- Define SLOs (latency p95, error rate) and fail the pipeline when thresholds breach

### Test Automation in CI/CD

- Every PR runs unit and integration tests — no exceptions, no manual gates for these
- E2E tests run on merge to main or on a scheduled cadence (nightly)
- Pipeline structure: build → unit tests → integration tests → deploy to staging → E2E/smoke
- Fast feedback: unit tests in <5 minutes, integration tests in <15 minutes
- Parallelize test execution — split by test category, not arbitrary batches
- Publish test results to Azure Test Plans or pipeline test analytics for trend tracking

### Deterministic Tests

- No random data — use seeded generators or fixed test data for reproducibility
- No clock dependencies — inject `IClock`/`TimeProvider` interfaces, don't call `DateTime.Now`
- No network calls in unit tests — mock HTTP clients, use `WireMock` for integration tests
- No shared mutable state between tests — each test is self-contained
- Tests produce the same result regardless of execution order or environment

### Naming Conventions

- **MethodName_Scenario_ExpectedResult:** `CalculateDiscount_ExpiredCoupon_ReturnsZero`
- **Given_When_Then:** `GivenExpiredCoupon_WhenCalculatingDiscount_ThenReturnsZero`
- Pick one convention per project and enforce it — consistency trumps preference
- Test class name mirrors the class under test: `OrderServiceTests` tests `OrderService`
- Descriptive names replace comments — a failing test name should explain what broke

### Test Data Management

- **Factories:** `TestOrderFactory.Create(status: "shipped")` — readable, maintainable defaults
- **Builders:** fluent API for complex objects — `new OrderBuilder().WithItems(3).Build()`
- **Fixtures:** shared setup for expensive resources (database connections, HTTP servers)
- Never hardcode IDs, dates, or magic strings — use constants or factory defaults
- Use containers (Testcontainers) for database-dependent tests — real engine, isolated instance

### Quality Gates

| Gate | Threshold | Enforcement |
|------|-----------|-------------|
| **Unit test pass rate** | 100% | PR blocked on failure |
| **Code coverage** | ≥70% (branch coverage preferred) | Warning at 60%, block at 50% |
| **Integration test pass rate** | 100% | Merge to main blocked on failure |
| **Static analysis** | Zero critical/high findings | PR blocked |
| **Security scanning** | Zero critical vulnerabilities | PR blocked |
| **Compliance checks** | Policy-compliant resources only | Deploy blocked |

- Quality gates are automated — no manual approvals for unit/integration results
- Track trends over time: coverage drift, flaky test rate, mean time to fix failing tests
- Review gate thresholds quarterly — tighten as the codebase stabilizes

### Contract Testing

- Verify API consumers and producers agree on schemas, not just that endpoints return 200
- Use Pact, Spectral, or OpenAPI schema validation for consumer-driven contract tests
- Run contract tests in CI on both consumer and provider sides
- Prevents breaking changes from reaching integration environments

### Shift-Left Security Testing

- Static Application Security Testing (SAST) in PR pipelines — catch vulnerabilities before merge
- Dependency scanning (Dependabot, Snyk, GitHub Advanced Security) on every build
- Secret scanning in pre-commit hooks and CI — block commits containing credentials
- Security tests are tests — they run automatically, not as a separate manual phase

### References

- [Architecture strategies for testing — Azure Well-Architected](https://learn.microsoft.com/azure/well-architected/operational-excellence/testing)
- [Recommendations for reliability testing — Azure Well-Architected](https://learn.microsoft.com/azure/well-architected/reliability/testing-strategy)
- [Recommendations for performance testing — Azure Well-Architected](https://learn.microsoft.com/azure/well-architected/performance-efficiency/performance-test)
- [Testing considerations for sustainable workloads](https://learn.microsoft.com/azure/well-architected/sustainability/sustainability-testing)
