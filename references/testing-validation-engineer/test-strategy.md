## Test Strategy

### The Test Pyramid

The test pyramid defines the optimal distribution of test types. More tests at the base (fast, cheap) and fewer at the top (slow, expensive).

```
    /  E2E  \        10% — Full system, slowest, most brittle
   /----------\
  / Integration \    20% — Real dependencies, moderate speed
 /----------------\
/    Unit Tests    \ 70% — Fast, isolated, mock dependencies
--------------------
```

### Target Ratio

| Level | Percentage | Characteristics |
|-------|-----------|-----------------|
| **Unit** | 70% | Fast (<100ms), isolated, mock all dependencies, test business logic |
| **Integration** | 20% | Real dependencies (DB, APIs), slower (seconds), test component interaction |
| **E2E** | 10% | Full system, slowest (minutes), most brittle, test critical user flows only |

### Unit Tests

- Test a single unit of logic in isolation
- Mock all external dependencies (databases, APIs, file systems)
- Should run in milliseconds — if a unit test is slow, it is not a unit test
- Focus on business rules, edge cases, error handling
- Use AAA pattern: **Arrange**, **Act**, **Assert**
- One logical assertion per test — test one behavior at a time
- Name tests descriptively: `MethodName_Scenario_ExpectedResult`

### Integration Tests

- Use real dependencies where possible (real database, real message queue)
- Test component boundaries and interaction contracts
- Slower than unit tests — accept seconds, not minutes
- Use test containers or ephemeral environments for infrastructure dependencies
- Validate serialization, network calls, authentication flows
- Clean up state between tests to prevent cross-test contamination

### E2E Tests

- Test complete user workflows through the full system stack
- Reserve for critical business paths only — login, checkout, core transactions
- Accept that these tests will be slower and more brittle
- Run in a production-like environment with realistic data
- Invest in stability over coverage — a few reliable E2E tests beat many flaky ones
- Use retry logic for infrastructure-related flakiness, not logic errors

### Shift Left — Test Early

- Run unit tests on every commit and pull request
- Run integration tests in CI before merge
- Run E2E tests in staging after deployment
- Catch defects as early as possible — cost of fixing grows exponentially with time
- Static analysis and linting are the earliest quality gate

### Test in CI, Not Just Locally

- If tests only run on developer machines, they are not real tests
- CI is the single source of truth for test results
- Ensure CI test environments match production configuration
- Tests that pass locally but fail in CI indicate environment coupling
- Pipeline should fail fast — run fastest tests first
- Parallelize test execution where possible to keep feedback loops short

### Anti-Patterns

- **Testing theater** — high coverage on trivial code (getters, DTOs) while ignoring complex logic
- **Ice cream cone** — more E2E tests than unit tests (inverted pyramid)
- **Testing implementation** — tests break when refactoring without behavior change
- **Shared mutable state** — tests depend on order or other tests' side effects
- **Ignoring test failures** — marking tests as "known failures" instead of fixing them
