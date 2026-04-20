## Testing Anti-Patterns

### Testing Implementation Details Instead of Behavior

- Tests coupled to internal method names, private state, or implementation structure
- Refactoring breaks tests even though behavior hasn't changed
- **Fix:** test public API contracts and observable outcomes, not how the code does it
- Ask: "Would a user care if this changed?" — if no, don't test it directly

### Flaky Tests

- Tests that pass or fail randomly due to timing, ordering, or external dependencies
- Common causes: race conditions, shared state, network calls, clock-dependent logic
- Flaky tests erode trust — teams start ignoring red builds
- **Fix:** isolate dependencies, use deterministic data, avoid `Thread.Sleep`/`Task.Delay` as synchronization
- Quarantine flaky tests immediately — track and fix, never normalize them

### Inverted Test Pyramid

- All end-to-end tests, no unit tests — the "ice cream cone" anti-pattern
- E2E tests are slow, brittle, and expensive to maintain
- Failures give vague signals — hard to isolate what broke
- **Fix:** 70% unit / 20% integration / 10% E2E — fast feedback at the base, confidence at the top

### Mocking Everything

- Tests that verify mock interactions instead of real behavior
- Every dependency stubbed — test proves nothing about actual integration
- Changing mock setup is more work than writing the real implementation
- **Fix:** mock only external boundaries (APIs, databases, file systems), use fakes for complex collaborators
- Integration tests should use real dependencies where feasible

### Test Data Coupling

- Tests depend on specific database rows, IDs, or environment state
- Tests fail in CI because the expected data doesn't exist
- Shared test data causes cascading failures when one test mutates it
- **Fix:** each test creates its own data (factories/builders), tears it down after
- Use in-memory databases or containers for isolation

### No Assertion Tests

- Tests that execute code but never verify outcomes
- Pass by default — they only fail on unhandled exceptions
- Create false confidence in coverage numbers
- **Fix:** every test must assert on at least one expected outcome
- Use linting rules to detect assertion-free tests (e.g., `xunit.analyzers`, `MSTest0037`)

### Copy-Paste Test Code

- Duplicated setup, duplicated assertions, duplicated teardown across tests
- Changing a pattern requires editing dozens of test methods
- **Fix:** extract shared setup into fixtures, builders, or base classes
- Use parameterized/data-driven tests for variations of the same scenario

### Ignoring Failing Tests

- Commenting out or `[Ignore]`/`[Skip]` instead of fixing
- "We'll come back to it" — but nobody does
- Test suite gradually becomes unreliable decoration
- **Fix:** failing tests are bugs — file a work item, fix in current sprint
- Track `[Ignore]` count as a quality metric; set a maximum threshold

### Testing Only the Happy Path

- All tests pass valid inputs and expect success
- No coverage for null inputs, boundary values, authorization failures, or timeout scenarios
- Bugs discovered in production instead of tests
- **Fix:** for every happy-path test, write at least one negative/edge-case test
- Use equivalence partitioning and boundary value analysis

### Late Performance Testing

- Performance tests only run before release — too late to fix architecture issues
- Load problems discovered in staging require expensive redesigns
- **Fix:** establish baselines early, run performance regression tests in CI
- Shift-left: catch N+1 queries, missing indexes, and memory leaks during development
- Azure Load Testing can integrate into PR pipelines for early detection

### No Infrastructure Validation

- Deploying application code without verifying infrastructure resources exist
- Missing storage accounts, misconfigured networking, wrong SKUs discovered at runtime
- **Fix:** smoke tests post-deployment that validate resource existence and configuration
- Use `terraform plan`, Bicep what-if, or Azure policy compliance checks before deploy
- ARM template validation (`az deployment group validate`) doesn't catch all errors — supplement with integration tests

### Undocumented Environment Requirements

- Tests require specific environment setup that isn't documented or automated
- New team members can't run tests without tribal knowledge
- CI/CD environment differs from local — tests pass locally, fail in pipeline
- **Fix:** all test prerequisites defined in code (Docker Compose, setup scripts, README)
- Use containerized test environments for reproducibility

### References

- [Performance antipatterns for cloud applications — Azure Architecture](https://learn.microsoft.com/azure/architecture/antipatterns/)
- [Architecture strategies for testing — Azure Well-Architected](https://learn.microsoft.com/azure/well-architected/operational-excellence/testing)
- [MSTest usage rules — .NET Testing](https://learn.microsoft.com/dotnet/core/testing/mstest-analyzers/usage-rules)
