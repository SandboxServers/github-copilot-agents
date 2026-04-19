## Test Strategy Design

### Minimum Viable Test Strategy
For any deployment, validate at minimum:
1. **Can the service start?** (smoke test)
2. **Can it handle expected load?** (basic load test)
3. **Is it configured correctly?** (infrastructure validation)
4. **Does it meet security requirements?** (policy compliance)
5. **Can it recover from failure?** (basic chaos test)

### Test Maintenance
- **Delete tests that don't catch bugs** — if a test has never failed meaningfully, question its value
- **Fix flaky tests immediately** — flaky tests erode trust in the entire suite
- **Keep test execution fast** — slow test suites get bypassed
- **Review test coverage quarterly** — what's changed that isn't being tested?
- **Automate everything** — manual test steps are test debt
