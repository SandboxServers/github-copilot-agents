## Testing Gotchas

### Terraform Test — Provider Authentication

- `terraform test` requires valid provider authentication even for plan-only tests
- Tests that only assert on plan output still need credentials to initialize providers
- CI pipelines must configure service principal or managed identity before running `terraform test`
- Use environment variables (`ARM_CLIENT_ID`, `ARM_TENANT_ID`, etc.) — don't embed credentials in test files
- Partial backend configs help isolate test state from production state files

### Bicep — No Built-In Test Framework

- Bicep has no native `test` command like Terraform
- Use PowerShell/Pester with `New-AzDeployment -WhatIf` for deployment validation
- Pester tests assert on what-if output to verify expected resource changes
- ARM template validation (`az deployment group validate`) catches schema errors but misses runtime issues
- Supplement with post-deployment smoke tests that hit actual resource endpoints

### Azure Load Testing — JMeter Differences

- Azure Load Testing uses Apache JMeter under the hood but has subtle format differences
- Some JMeter plugins are not supported — check the compatibility list before authoring scripts
- JMX files must be self-contained; external JAR dependencies require explicit upload
- Thread group ramp-up behavior can differ between local JMeter and Azure Load Testing execution
- CSV data files must be uploaded separately and referenced by filename only (no paths)

### Integration Test Environment Cleanup

- Creating test resources is easy; cleaning them up reliably is hard
- Failed tests may leave orphaned resources — storage accounts, databases, resource groups
- **Always** use resource tagging (e.g., `environment: test`, `ttl: 2h`) for cleanup automation
- Schedule cleanup scripts (Azure Automation, Logic Apps) to sweep tagged resources
- Use dedicated test subscriptions with cost alerts to catch runaway resources

### Pester — Assertion Count ≠ Test Quality

- High assertion counts don't mean meaningful test coverage
- 100 assertions checking `$result -ne $null` add no value
- Quality indicators: assertions on specific values, error conditions, and edge cases
- Use `Should -BeExactly`, `Should -Throw`, `Should -HaveCount` for precise validation
- Measure what scenarios are covered, not how many `Should` statements exist

### Parallel Test Execution — Shared State

- Parallel test runs sharing databases, files, or global variables cause intermittent failures
- Test A writes data, Test B reads stale state, Test C deletes it — non-deterministic outcomes
- Solutions: isolated databases per test (containers), unique resource prefixes, transaction rollback
- In xUnit, `IClassFixture` shares within a class; `ICollectionFixture` shares across classes — know the scope
- Azure DevOps parallel jobs share nothing by default, but shared test databases break this isolation

### Mock vs Stub vs Fake — Wrong Test Double

- **Mock:** records calls, verifies interactions (use for outbound side-effects like sending email)
- **Stub:** returns canned data (use for dependencies that provide input to the system under test)
- **Fake:** working implementation with shortcuts (use for databases, file systems with in-memory alternatives)
- Using mocks where stubs suffice leads to brittle tests coupled to call sequences
- Over-mocking: if you mock more than two dependencies, the test may be testing the mocks

### ARM Validation Limitations

- `az deployment group validate` checks template syntax and basic parameter validation
- It does NOT check: quota limits, naming conflicts, RBAC permissions, or cross-resource dependencies
- Deployments can pass validation but fail during actual provisioning
- **Supplement with:** what-if deployments, policy compliance pre-checks, and integration tests
- Bicep linter (`bicep build`) catches some issues ARM validation misses

### Test Coverage Metrics Are Misleading

- 80% code coverage does not mean 80% of logic is tested
- Coverage measures lines executed, not correctness of assertions
- A test that calls a method without asserting anything still counts as "covered"
- Branch coverage is more meaningful than line coverage — ensures all conditional paths tested
- Use coverage as a floor (e.g., minimum 70%), not a ceiling or primary quality metric

### Load Testing — Test Data Matters More Than Volume

- 10,000 requests with identical payloads doesn't simulate real load
- Realistic distributions: varied user IDs, different payload sizes, mixed read/write ratios
- Cache behavior changes drastically with realistic vs synthetic data
- Use parameterized JMeter tests with CSV data sets that reflect production patterns
- Include think time between requests — zero-delay hammering doesn't reflect user behavior

### Pipeline Test Timeouts

- Default test timeouts in CI pipelines are often too short for integration tests
- Tests pass locally (fast disk, warm caches) but time out in CI (shared agents, cold starts)
- Set explicit timeouts per test category: unit (30s), integration (5min), E2E (15min)
- Azure Pipelines: use `timeoutInMinutes` at job level, not just overall pipeline timeout

### References

- [Architecture strategies for testing — Azure Well-Architected](https://learn.microsoft.com/azure/well-architected/operational-excellence/testing)
- [Performance antipatterns — Azure Architecture](https://learn.microsoft.com/azure/architecture/antipatterns/)
- [Azure Load Testing documentation](https://learn.microsoft.com/azure/load-testing/)
