## Test Automation

### CI Integration Patterns

- Run tests as a first-class pipeline step — not an optional add-on
- Fail the pipeline immediately on test failure — no "continue on error" for test stages
- Publish test results in standard formats (JUnit XML, NUnit XML, TRX) for dashboard consumption
- Separate test stages by type: unit tests → integration tests → E2E tests
- Run the fastest tests first to provide rapid feedback
- Cache test dependencies (packages, containers) to speed up execution

### Test Parallelization

- Split test suites across multiple agents to reduce total execution time
- Partition by test file, test class, or custom sharding strategy
- Ensure tests are independent — parallel execution exposes hidden dependencies
- Use test frameworks' built-in parallel capabilities (xUnit collections, pytest-xdist, Jest workers)
- Monitor for resource contention when parallelizing integration tests (database connections, ports)
- Balance shard sizes — uneven distribution wastes agent capacity

### Flaky Test Management

Flaky tests erode confidence in the entire test suite. Treat them as high-priority bugs.

| Strategy | When to Use |
|----------|------------|
| **Quarantine** | Immediately isolate flaky tests into a separate suite that does not block the pipeline |
| **Retry** | Allow 1-2 retries for infrastructure-related flakiness (network, container startup) |
| **Fix** | Root-cause and fix within one sprint — quarantine is temporary, not permanent |
| **Delete** | If a flaky test cannot be fixed and provides no value, remove it |

- Track flaky test rate as a team metric — aim for <1% flaky rate
- Investigate patterns: timing-dependent, order-dependent, environment-dependent
- Never add `[Retry]` or `@retry` as a permanent solution — retries mask real bugs

### Test Reporting and Dashboards

- Aggregate test results across all pipeline stages into a single dashboard
- Track key metrics: pass rate, execution time, flaky rate, coverage trend
- Configure alerts for test suite degradation (pass rate drops, execution time increases)
- Link test failures to specific commits and pull requests for fast root-cause identification
- Retain historical results for trending — "are we getting better or worse?"

### Test Environment Provisioning

- **Ephemeral environments** — spin up per-PR or per-branch test environments, destroy after tests complete
- Use infrastructure-as-code (Terraform, Bicep) for reproducible environment setup
- Containerize dependencies (databases, queues, caches) with Docker Compose or Testcontainers
- Environment provisioning should be fully automated — no manual setup steps
- Budget for environment costs — ephemeral environments can be expensive at scale
- Clean up orphaned environments on a schedule to prevent cost leaks

### Artifact Management

- Store test reports, coverage reports, and screenshots as pipeline artifacts
- Attach failure screenshots and logs to test result records for debugging
- Version test artifacts alongside build artifacts for traceability
- Retain artifacts for a defined period (30-90 days) to support incident investigation
- Use artifact storage (Azure Artifacts, S3, blob storage) — not pipeline logs

### Test Result Trending

- Compare test results across builds to identify regressions early
- Track execution time trends — gradually slowing tests indicate design issues
- Monitor test count growth — ensure new features come with new tests
- Correlate test failures with infrastructure changes (new dependency versions, platform updates)
- Report trends weekly to the team — visibility drives accountability
