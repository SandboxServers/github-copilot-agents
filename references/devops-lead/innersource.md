## InnerSource — Sharing Automation Across Teams

### InnerSource Principles

InnerSource applies open-source practices inside the organization:
- **Discoverability**: Shared repos are findable — internal catalog or search
- **Contribution model**: Anyone can propose changes via pull requests
- **Maintainers**: Clear ownership — someone reviews PRs and drives the roadmap
- **Documentation**: README, CONTRIBUTING.md, architecture decisions recorded
- **Versioning**: Consumers pin to versions — maintainers don't break consumers without warning

### InnerSource for CI/CD

- **Shared template repos**: Organization-wide pipeline templates that any team can consume and contribute to
- **Composite actions library**: Internal marketplace of reusable GitHub Actions
- **IaC module registry**: Shared Terraform modules or Bicep modules in internal registry
- **Fork workflow**: Teams fork shared repos, customize, and contribute improvements back upstream

### Avoiding the Monolith

- **Federated ownership**: Teams own their templates, shared ones have explicit maintainers
- **Loose coupling**: Templates accept parameters, don't hard-code assumptions
- **Version pinning**: Consumers choose when to upgrade — no forced updates
- **Clear boundaries**: Shared ≠ mandated — adoption should be driven by value, not decree
