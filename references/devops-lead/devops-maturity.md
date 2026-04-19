## DevOps Maturity Assessment

### Assessment Dimensions

1. **Source control**: From no version control → Git with branching strategy → trunk-based development with feature flags
2. **Build automation**: From manual builds → CI on commit → fast feedback with parallelized tests
3. **Test automation**: From manual testing only → unit tests in CI → full test pyramid (unit, integration, e2e, performance)
4. **Deployment automation**: From manual deployment → scripted deployment → continuous deployment with canary/blue-green
5. **Monitoring and feedback**: From no monitoring → basic health checks → full observability with DORA tracking
6. **Culture and collaboration**: From siloed teams → shared responsibility → blameless postmortems and continuous improvement
7. **Security integration**: From security as a gate → shift-left scanning → DevSecOps with automated policy enforcement

### Prescriptive Improvement — Not Generic Advice

When assessing a team, prescribe **specific next steps**, not platitudes:
- Instead of "improve your CI" → "Add unit test execution to your PR build with a minimum 60% coverage gate"
- Instead of "automate more" → "Replace the manual deployment to staging with an Azure Pipelines release stage triggered on PR merge"
- Instead of "shift left on security" → "Add Trivy container scanning to your Docker build step and fail the pipeline on critical CVEs"

### The CI/CD Product Mindset

CI/CD is a product with internal customers:
- **Customer research**: Talk to development teams about friction points
- **Backlog**: Prioritize platform improvements like any product backlog
- **SLAs**: Pipeline availability, queue wait times, build duration targets
- **Release notes**: Communicate template changes, new features, deprecations
- **Support channels**: Where do teams go when the pipeline breaks?
- **Feedback loops**: Regular surveys, usage telemetry, retrospectives
