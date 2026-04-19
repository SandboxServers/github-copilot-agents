## Golden Paths and Developer Experience

### What Makes a Golden Path

A golden path (paved road) is the supported, recommended way to accomplish a common task. It makes the right thing the easy thing.

**Characteristics of effective golden paths:**
- **Opinionated but not locked**: Provide a default that works for 80%+ of cases
- **Self-service**: Developers can use them without filing a ticket
- **Documented**: Clear README, examples, and troubleshooting
- **Tested**: The path itself is validated — templates, pipelines, configurations
- **Maintained**: Someone owns it and keeps it current
- **Observable**: Usage is tracked so you know adoption and can measure impact

### Golden Path Examples for CI/CD

- **Starter pipeline template**: New repo gets a working CI/CD pipeline in minutes
- **Environment promotion**: Dev → staging → production with approval gates
- **Infrastructure provisioning**: Self-service Terraform/Bicep modules with pipeline integration
- **Security scanning**: SAST/DAST/dependency scanning integrated into every pipeline
- **Container build**: Dockerfile → build → scan → push to registry → deploy

### The Deviation Policy

- Supported paths are free — the platform team maintains them
- Deviations are allowed but owned by the deviating team
- Track deviations — recurring deviations signal a gap in the golden path
- Evaluate popular deviations for inclusion in the supported set
