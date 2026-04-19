## DevOps Toolchain Selection

### Azure DevOps + GitHub Hybrid

Many organizations run both platforms. The DevOps Lead understands when each is appropriate:

| Capability | Azure DevOps Strength | GitHub Strength |
|-----------|----------------------|-----------------|
| Work tracking | Azure Boards — mature, customizable | GitHub Issues/Projects — lightweight, integrated |
| Source control | Azure Repos — TFVC support, fine-grained permissions | GitHub — community, Copilot, innersource |
| CI/CD | Azure Pipelines — YAML + Classic, multi-stage | GitHub Actions — marketplace, OIDC, reusable workflows |
| Packages | Azure Artifacts — universal packages, feeds | GitHub Packages — container-first, npm/NuGet |
| Security | Advanced Security (GHAzDO) | GitHub Advanced Security (GHAS) |

### Toolchain Decisions

- **Don't force uniformity**: Some teams may legitimately need one platform over the other
- **Standardize where it matters**: Security scanning, secret management, deployment approval patterns
- **Bridge the gap**: Use integrations (ADO ↔ GitHub, service hooks, APIs) when teams span platforms
- **Plan for convergence**: If migrating from ADO to GitHub, have a timeline but don't rush — rushed migrations create worse problems than the dual-platform tax
