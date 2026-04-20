## ALM (Application Lifecycle Management)

### Solutions: Managed vs Unmanaged

| Aspect | Unmanaged | Managed |
|---|---|---|
| **Used in** | Development environments | Test, staging, production |
| **Editable** | Yes — components can be modified | No — read-only in target environment |
| **Uninstall** | Components remain after removal | Clean removal of all components |
| **Layering** | Active customization layer | Managed layer — conflicts resolved by layering rules |

- Always develop in unmanaged solutions, deploy as managed
- Never build directly in the default environment outside a solution

### Solution Components
- Tables, columns, relationships, views, forms, charts, dashboards
- Canvas apps, model-driven apps, custom pages
- Cloud flows, business process flows, desktop flow definitions
- Security roles, connection references, environment variables
- Web resources, plug-in assemblies, custom connectors
- **Segment solutions** — include only changed components, not entire tables
- Use consistent publisher prefix (not "new_" or "cr" — those are reserved/default)

### Export and Import
- Export from dev as managed solution (.zip)
- Import to test/prod — system resolves dependencies and applies changes
- **Pre-import validation** checks for missing dependencies and conflicts
- Upgrade vs update: upgrade removes components deleted from solution; update does not
- Always use upgrade for production deployments to keep environments clean

### Environment Variables
- Store configuration values that differ between environments (URLs, feature flags)
- Types: text, number, JSON, yes/no, data source
- Set definition in solution, set current value per environment (not in solution itself)
- **Secret environment variables** — reference Azure Key Vault secrets for sensitive data
- Critical for ALM — never hardcode environment-specific values in apps or flows

### Connection References
- Decouple connections from solutions so they can be rebound per environment
- Solution stores the reference — admin provides actual connection at import time
- Must configure for each target environment during deployment
- Map to service principal connections for unattended/automated scenarios

### Power Platform Build Tools (Azure DevOps)
- Official ADO tasks: export solution, import solution, pack/unpack, check solution
- Solution Packager unpacks solutions to human-readable files for source control
- Deployment settings files (JSON) map connection references and env vars per target
- Create pipelines: export → unpack → commit → PR → build managed → deploy

### GitHub Actions for Power Platform
- Same capabilities as Build Tools but for GitHub-based workflows
- Actions: export-solution, import-solution, pack-solution, unpack-solution
- Integrate with GitHub pull requests for solution validation
- Use solution checker action for automated quality gates

### Pipelines for Power Platform (Built-In)
- Native deployment pipeline: Dev → Test → Production within Power Platform
- Configured by admins in Deployment Pipeline Configuration app
- Makers deploy with clicks — no DevOps tooling required
- Pre-flight checks validate dependencies before deployment
- Connection references and environment variables prompted during deployment
- Same artifact passes through all stages (no tampering between stages)
- Cannot skip stages — enforces sequential promotion
- Supports delegated deployments with approval gates

### Versioning Strategy
- Use semantic versioning for solutions (major.minor.patch)
- Increment major for breaking changes, minor for features, patch for fixes
- Tag source control commits with solution version numbers
- Maintain changelog of what changed between versions
- Solution Packager enables meaningful diffs in source control
