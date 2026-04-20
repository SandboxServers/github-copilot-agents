# Azure Terraform Author — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, best-practices, AVM stance update, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Style Guide | [style-guide.md](style-guide.md) | **ALWAYS load first.** Naming, formatting, file layout, commenting conventions |
| Provider Landscape | [provider-landscape.md](provider-landscape.md) | When choosing between azurerm vs azapi, provider config, authentication |
| Module Architecture | [module-architecture.md](module-architecture.md) | When designing modules — directory structure, composition (AVM is reference-only) |
| State Management | [state-management.md](state-management.md) | When working with state backends, workspaces, locking, import, drift |
| Code Patterns | [code-patterns.md](code-patterns.md) | When using validation, lifecycle, dynamic blocks, moved blocks, conditionals |
| Testing | [testing.md](testing.md) | When writing tests — .tftest.hcl, mock providers, tflint, checkov, CI integration |
| Common Gotchas | [common-gotchas.md](common-gotchas.md) | When troubleshooting replacement triggers, ignore_changes, RBAC propagation, timeouts |
| Managed Identity & RBAC | [managed-identity.md](managed-identity.md) | When implementing managed identity, OIDC federation, role assignments, data plane RBAC |
| Code Quality | [code-quality.md](code-quality.md) | When reviewing PRs — anti-patterns, checklists, security scanning, policy-as-code |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing code for common Terraform/Azure mistakes and how to fix them |
| Best Practices | [best-practices.md](best-practices.md) | When writing new Terraform — module structure, validation, lifecycle, tagging, outputs |
| Resource Adoption | [resource-adoption.md](resource-adoption.md) | When importing existing Azure resources into Terraform management |

## Related Skills

| Skill | When to Load |
|-------|-------------|
| `terraform-style-guide` | Terraform naming, formatting, file organization, and PR review checklist |
| `naming-conventions` | Cross-language naming standards including Terraform conventions |
| `iac-review` | Infrastructure code review checklist (Terraform, Bicep, K8s) |
| `infrastructure-testing` | Terraform testing pyramid — static analysis, plan validation, Terratest |
