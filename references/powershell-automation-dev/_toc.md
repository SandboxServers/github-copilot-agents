# PowerShell Automation Developer — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Style Guide | [style-guide.md](style-guide.md) | **Always load first** — naming, commenting, error handling, logging, idempotency, and security rules |
| Best Practices | [best-practices.md](best-practices.md) | When writing new functions or reviewing code — advanced function template, error handling, logging, idempotency, credentials, cross-platform, testing, modules |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing code or debugging — Write-Host misuse, missing CmdletBinding, aliases in scripts, hardcoded credentials, Invoke-Expression, no ShouldProcess |
| Gotchas | [gotchas.md](gotchas.md) | When debugging unexpected behavior — array unrolling, return semantics, comparison operators with arrays, scope inheritance, $null comparison order, double-hop auth |
| PS7 Language Features | [ps7-language-features.md](ps7-language-features.md) | When using PS7-specific features — ternary, null-coalescing, ForEach-Parallel, pipeline chains, clean block, $PSStyle |
| Error Handling | [error-handling.md](error-handling.md) | When designing error handling — ErrorAction, try/catch, ThrowTerminatingError, trap, retry patterns, parallel error handling |
| Module Ecosystem | [module-ecosystem.md](module-ecosystem.md) | When working with Graph SDK, Az modules, Exchange, Teams, PnP, or SharePoint Online Management Shell |
| Authentication Patterns | [authentication-patterns.md](authentication-patterns.md) | When implementing managed identity, certificate auth, workload identity federation, token acquisition, or SecretManagement |
| Execution Environments | [execution-environments.md](execution-environments.md) | When choosing Azure Automation, Functions, GitHub Actions, Azure Pipelines, containers, or Hybrid Workers |
| Azure Functions (PS) | [azure-functions-powershell.md](azure-functions-powershell.md) | When writing PowerShell Azure Functions — triggers, bindings, project structure, profile.ps1, requirements.psd1, durable functions, deployment |
| REST API Patterns | [rest-api-patterns.md](rest-api-patterns.md) | When calling REST/Graph APIs — Invoke-RestMethod, pagination, retry with backoff, throttling (429), batch requests, authentication, error handling |
| Performance Optimization | [performance-optimization.md](performance-optimization.md) | When optimizing collections, string building, hashtable lookups, file I/O, caching, or parallel processing |
| Pester Testing | [pester-testing.md](pester-testing.md) | When writing Pester v5 tests — structure, assertions, mocking, data-driven tests, TestDrive, InModuleScope, CI/CD |
| Module Development | [module-development.md](module-development.md) | When building PowerShell modules — structure, manifest, versioning, publishing, CI/CD pipeline |
