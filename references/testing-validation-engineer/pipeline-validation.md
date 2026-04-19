## Pipeline Validation

### What to Validate in CI/CD
1. **Does the pipeline do what the team thinks?** — review pipeline YAML against expected behavior
2. **Are tests actually running?** — verify test tasks execute and results are consumed
3. **Are there gaps between tested and deployed?** — if the pipeline deploys to 3 environments but only tests in 1, that's a gap
4. **Are artifacts consistent?** — same build artifact should flow through all environments
5. **Are secrets handled correctly?** — no secrets in logs, proper variable group usage
6. **Do quality gates block bad deployments?** — test failures should prevent promotion

### Pipeline Test Integration Points
| Stage | Tests to Run |
|---|---|
| **PR validation** | Lint, static analysis, unit tests (L0/L1) |
| **CI build** | Unit tests, integration tests (L2), security scanning |
| **Staging deploy** | Smoke tests, functional tests (L3), load tests |
| **Production deploy** | Smoke tests, synthetic monitoring, canary validation |
| **Post-deploy** | Integration tests (L4), chaos testing, performance baseline |
