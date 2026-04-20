# PowerShell Automation Developer — Knowledge Index

> **Last Updated:** 2025-07-17 — All chunks rewritten with MCP research (Context7 + MS Learn). Style guide added.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Style Guide | [style-guide.md](style-guide.md) | **Always load first** — naming, commenting, error handling, logging, idempotency, and security rules |
| PS7 Language Features | [ps7-language-features.md](ps7-language-features.md) | When using PS7-specific features — ternary, null-coalescing, ForEach-Parallel, pipeline chains, clean block, $PSStyle |
| Error Handling | [error-handling.md](error-handling.md) | When designing error handling — ErrorAction, try/catch, ThrowTerminatingError, trap, retry patterns, parallel error handling |
| Module Ecosystem | [module-ecosystem.md](module-ecosystem.md) | When working with Graph SDK, Az modules, Exchange, Teams, PnP, or SharePoint Online Management Shell |
| Authentication Patterns | [authentication-patterns.md](authentication-patterns.md) | When implementing managed identity, certificate auth, workload identity federation, token acquisition, or SecretManagement |
| Execution Environments | [execution-environments.md](execution-environments.md) | When choosing Azure Automation, Functions, GitHub Actions, Azure Pipelines, containers, or Hybrid Workers |
| Performance Optimization | [performance-optimization.md](performance-optimization.md) | When optimizing collections, string building, hashtable lookups, file I/O, caching, or parallel processing |
| Pester Testing | [pester-testing.md](pester-testing.md) | When writing Pester v5 tests — structure, assertions, mocking, data-driven tests, TestDrive, InModuleScope, CI/CD |
| Module Development | [module-development.md](module-development.md) | When building PowerShell modules — structure, manifest, versioning, publishing, CI/CD pipeline |
