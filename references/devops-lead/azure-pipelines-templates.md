## Azure Pipelines — Template Architecture at Scale

### Template Types

1. **Includes templates**: Insert reusable content (steps, jobs, stages, variables) into a pipeline. Functions like an include directive — content is inserted where referenced.
2. **Extends templates**: Control and define the schema for what is allowed in a pipeline. Enforces security, compliance, and organizational standards. The pipeline must follow the structure the extends template defines.

### Template Architecture Patterns

- **Shared template repository**: Central repo containing organizational templates, referenced across all project repos
- **Extends for governance**: Use extends templates to enforce required security scans, approval gates, and artifact signing
- **Includes for reuse**: Use includes templates for common build steps, test configurations, and deployment patterns
- **Variable templates**: Centralize environment-specific configuration in variable template files
- **Parameterized templates**: Use template parameters with type safety for customization without forking

### Template Limits

- Maximum 100 separate YAML files included (directly or indirectly)
- Maximum 100 levels of template nesting
- Maximum 20 MB memory consumed while parsing YAML

### Managing Templates at Scale (200+ Pipelines)

- **Version your templates**: Use Git tags or branches for template versioning so consumers can pin to a known-good version
- **Semantic versioning**: Breaking changes get a major version bump; consumers opt in to upgrades
- **Template testing**: Test templates in isolation before rolling out — a broken shared template breaks every consumer
- **Rolling updates**: When rolling out breaking changes, provide a migration window — don't force-update everyone simultaneously
- **Documentation**: Every template needs a README — parameters, expected behavior, examples
- **Deprecation policy**: Announce deprecation, provide migration path, set a sunset date, then remove
