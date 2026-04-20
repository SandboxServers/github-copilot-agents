## Quality Gates

### Pipeline Quality Gates

Quality gates are automated checkpoints in the CI/CD pipeline that block promotion when standards are not met. Every gate must be automated — manual checks do not scale.

### Code Coverage

- **Minimum threshold: 80%** line coverage for application code
- Measure branch coverage in addition to line coverage for meaningful metrics
- Exclude auto-generated code, DTOs, and configuration classes from coverage calculation
- Coverage trending — alert when coverage drops between builds, not just below threshold
- Coverage is necessary but not sufficient — high coverage with weak assertions is testing theater
- Enforce at PR level — PRs that reduce coverage below threshold are blocked

### Static Analysis

| Tool | Purpose | Integration |
|------|---------|-------------|
| **SonarQube** | Code quality — bugs, code smells, technical debt | PR decoration, quality gate |
| **CodeQL** | Semantic code analysis — security vulnerabilities | GitHub Advanced Security |
| **ESLint / Pylint** | Language-specific linting | Pre-commit hooks, CI |
| **PSScriptAnalyzer** | PowerShell static analysis | CI pipeline |
| **Bicep linter** | Infrastructure-as-code best practices | CI pipeline |

- Configure as PR checks — block merge on new critical or high issues
- Track technical debt over time — enforce that debt does not grow sprint over sprint
- Establish baseline and require improvement, not perfection on legacy code

### Security Scanning

| Tool | What It Scans | When |
|------|--------------|------|
| **Dependabot** | Dependency vulnerabilities (CVEs) | Continuous, PR alerts |
| **Snyk** | Dependencies + container images + IaC | PR validation, CI |
| **Trivy** | Container images, filesystems, IaC | CI pipeline |
| **OWASP ZAP** | Dynamic application security (DAST) | Staging deployment |
| **Microsoft Defender for DevOps** | Cross-pipeline security posture | Continuous |

- Block deployment on critical or high severity vulnerabilities
- Allow medium/low with documented exceptions and remediation timeline
- Scan container base images — not just application dependencies

### License Compliance

- Scan all dependencies for license compatibility
- Block copyleft licenses (GPL) in proprietary projects unless explicitly approved
- Maintain an allow-list of approved licenses (MIT, Apache-2.0, BSD)
- Automate license checks in CI — do not rely on manual review

### Test Pass Rate

- **100% test pass rate required** to proceed through any quality gate
- No "known failure" exceptions — fix or delete failing tests
- Flaky tests must be quarantined and fixed within one sprint
- Test results published to pipeline for visibility and trending

### Performance Regression

- Compare key metrics (P95 response time, throughput) against established baseline
- Block deployment if P95 degrades by more than 10% versus baseline
- Run performance tests in staging as part of the deployment pipeline
- Refresh baselines after intentional architecture or capacity changes

### Manual Approval Gates

- Required for production deployments in regulated environments
- Approvers review deployment summary: what changed, test results, risk assessment
- Time-boxed approvals — auto-reject if not approved within window
- Approval records retained for audit trail
- Minimize manual gates — automate everything that can be automated
