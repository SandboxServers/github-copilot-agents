## InnerSource — Sharing Automation Across Teams

InnerSource applies open-source collaboration practices within an organization. For DevOps teams, InnerSource is the mechanism that turns one team's automation into organizational capability without creating a centralized bottleneck.

### Why InnerSource for DevOps

Every team eventually solves similar problems — building containers, deploying to Kubernetes, scanning for vulnerabilities, managing Terraform state. Without InnerSource, each team reinvents these solutions independently. InnerSource creates a multiplier effect where good automation spreads organically.

### Shared Template Repositories

The primary InnerSource pattern for DevOps is the shared template repository. This contains pipeline templates, reusable workflow definitions, and composite actions that any team can consume.

A well-structured template repo includes:
- Pipeline templates organized by concern (build, test, deploy, scan)
- Clear input parameters with sensible defaults and documentation
- Example pipelines showing how to compose templates together
- Automated tests that validate templates work correctly
- A CHANGELOG tracking what changed between versions

### Contributing Guidelines

Every shared repository needs a CONTRIBUTING.md that answers:
- How to propose a new template or modification — issue first, then PR
- Code review expectations — who reviews, what the bar is, expected turnaround
- Testing requirements — contributors must include tests for new templates
- Style conventions — naming patterns, parameter conventions, documentation format
- How to handle breaking changes — deprecation notices, migration guides, version bumps

### Review Process

InnerSource PRs need clear review processes:
- **Maintainers** review all PRs to shared repos — they own quality and direction
- **Contributors** propose changes via fork-and-PR or branch-and-PR depending on access model
- Reviews should be timely — if PRs sit for weeks, contributors stop contributing
- Automated checks (linting, template validation, test runs) gate PRs before human review

### Discoverability

Shared automation is only valuable if teams can find it:
- **Internal catalog**: A README or wiki page listing all shared repos with descriptions and links
- **Search**: Consistent naming conventions so repos are findable (e.g., `shared-pipeline-templates`, `reusable-actions`)
- **Documentation**: Each template has a README explaining what it does, required inputs, and example usage
- **Onboarding**: New teams are pointed to shared resources during onboarding — discovery should not require tribal knowledge

### Ownership Model

- **Maintainers**: A small group (2–4 people) who own the repo, review PRs, manage releases, and set direction. Maintainers are explicitly listed in the repo
- **Contributors**: Anyone in the organization who submits PRs. Contributors do not need approval to fork or propose changes
- **Consumers**: Teams that use templates without contributing back. This is fine — not every consumer needs to be a contributor

### Versioning and Backward Compatibility

- Use semantic versioning — major versions for breaking changes, minor for new features, patch for fixes
- Consumers pin to a specific major version and opt into upgrades on their schedule
- Never force-update consumers — breaking changes require a new major version with a migration guide
- Maintain the previous major version with security fixes for a defined support window
- Tag releases in Git so consumers can reference exact versions in their pipelines
