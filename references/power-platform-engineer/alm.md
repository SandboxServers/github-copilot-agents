## ALM (Application Lifecycle Management)

### Solution-Based Development
1. **Always develop in solutions** — never build directly in the default environment outside a solution
2. **Use a consistent publisher** with unique prefix — avoid "new_" or "cr" (reserved/default)
3. **Segment solutions** — don't put everything in one giant solution
4. **Managed in production** — deploy managed solutions to test/prod, keep unmanaged only in dev

### Power Platform Pipelines (Native ALM)
- Built-in deployment pipeline: Dev → Test → Production
- Configured by admins in the Deployment Pipeline Configuration app
- Makers deploy with a few clicks from within their development environment
- **Pre-flight checks**: validates dependencies, missing components before deployment
- **Connection references + environment variables** prompted during deployment
- Solutions exported once, same artifact passes through all stages (no tampering possible)
- Can't skip stages (e.g., can't go from dev directly to prod)
- Supports **delegated deployments** with approvals
- **Service principal deployment** for automated/unattended scenarios
- Pipeline host should be a production environment
- Target environments (except dev) must be Managed Environments

### ALM with Azure DevOps / GitHub
- **Power Platform Build Tools** (Azure DevOps) or **GitHub Actions** for Power Platform
- Solution Packager — unpacks solutions to human-readable files for source control
- **Deployment settings files** — JSON config for connection references + environment variables per target
- ALM Accelerator (CoE toolkit) — reference implementation of full ALM pipeline with:
  - Branch creation, solution export/unpack, commit to Git
  - Pull request validation with Solution Checker
  - Managed solution build and deployment
  - Connection reference + environment variable configuration per environment

### Connection References + Environment Variables in ALM
- **Connection references**: abstract connections from solutions — rebound per environment at deployment
- **Environment variables**: store config values (URLs, flags) — set different values per environment
- **Never store sensitive data in environment variables** — use secret env vars with Azure Key Vault
- **Deployment settings file**: JSON mapping of connection reference logical names → connection IDs and env var names → values
