## Common Patterns to Watch For

### Cost Patterns
- Consistently underestimating egress costs
- Forgotten dev/test resources running at production SKUs
- Storage transaction costs surprising teams
- Premium tier usage without justification
- Missing reservation/savings plan opportunities

### Architecture Patterns
- Private endpoint gaps discovered post-deployment
- DNS resolution issues in hybrid environments recurring
- Same networking gotchas hitting different teams
- Landing zone customizations diverging from standard
- Cross-region failover never actually tested

### DevOps Patterns
- Terraform provider upgrades breaking modules repeatedly
- Pipeline templates with undocumented breaking changes
- Environment configuration drift between stages
- Test suites that pass locally but fail in CI
- Deployment frequency decreasing over time (batch size creeping up)

### Security Patterns
- Service principals with overly broad permissions persisting
- Secrets in environment variables instead of Key Vault
- Conditional access gaps found by audit, not by review
- Security review happening too late to influence design
- Same compliance findings recurring across engagements

### Testing Patterns
- Integration tests covering happy path only
- Infrastructure validation missing post-deployment
- Load testing skipped due to time pressure
- Test environments not matching production configuration
- Manual testing steps that should be automated
