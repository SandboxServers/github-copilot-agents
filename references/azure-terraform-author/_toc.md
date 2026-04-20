# Azure Terraform Author — Knowledge Index

> **Last Updated:** 2025-07-18 — All chunks replaced with MCP-researched content (Context7 + MS Learn).

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Style Guide | [style-guide.md](style-guide.md) | **ALWAYS load first.** Naming, formatting, file layout, commenting conventions |
| Provider Landscape | [provider-landscape.md](provider-landscape.md) | When choosing between azurerm vs azapi, provider config, authentication |
| Module Architecture | [module-architecture.md](module-architecture.md) | When designing modules — AVM patterns, directory structure, composition |
| State Management | [state-management.md](state-management.md) | When working with state backends, workspaces, locking, import, drift |
| Code Patterns | [code-patterns.md](code-patterns.md) | When using validation, lifecycle, dynamic blocks, moved blocks, conditionals |
| Testing | [testing.md](testing.md) | When writing tests — .tftest.hcl, mock providers, tflint, checkov, CI integration |
| Common Gotchas | [common-gotchas.md](common-gotchas.md) | When troubleshooting replacement triggers, ignore_changes, RBAC propagation, timeouts |
| Managed Identity & RBAC | [managed-identity.md](managed-identity.md) | When implementing managed identity, OIDC federation, role assignments, data plane RBAC |
| Code Quality | [code-quality.md](code-quality.md) | When reviewing PRs — anti-patterns, checklists, security scanning, policy-as-code |
