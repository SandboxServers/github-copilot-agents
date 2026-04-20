# GitHub Actions Architect — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Workflow Triggers | [workflow-triggers.md](workflow-triggers.md) | When configuring event triggers, filters, activity types, or debugging why a workflow didn't run |
| Expressions & Contexts | [expressions-contexts.md](expressions-contexts.md) | When writing expressions, accessing contexts, or debugging context availability errors |
| Reusable Workflows | [reusable-workflows.md](reusable-workflows.md) | When designing workflow_call patterns, output passing, nesting, or secret propagation |
| Composite Actions | [composite-actions.md](composite-actions.md) | When building composite, JavaScript, or Docker actions — inputs, outputs, versioning |
| Matrix Strategies | [matrix-strategies.md](matrix-strategies.md) | When designing matrix builds — include/exclude, dynamic matrices, failure handling |
| Security Hardening | [security-hardening.md](security-hardening.md) | When hardening workflows — permissions, SHA pinning, script injection, fork safety, supply chain |
| OIDC Deployments | [oidc-deployments.md](oidc-deployments.md) | When setting up OIDC for Azure, AWS, or GCP — federated credentials, trust policies |
| Caching & Artifacts | [caching-artifacts.md](caching-artifacts.md) | When optimizing with caching or managing artifacts — keys, scope, eviction, retention |
| Runners & Infrastructure | [runners-infrastructure.md](runners-infrastructure.md) | When selecting runners — hosted, larger, self-hosted, ARM, ARC scaling |
| Workflow Patterns | [workflow-patterns.md](workflow-patterns.md) | When implementing common patterns — CI, monorepo, release, deployment, rollback |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing workflows for common mistakes — monolithic files, hardcoded secrets, missing caching, supply chain risks |
| Gotchas | [gotchas.md](gotchas.md) | When debugging unexpected behavior — context availability, matrix failFast, nesting limits, fork secrets, artifact limits |
| Best Practices | [best-practices.md](best-practices.md) | When designing new workflows — OIDC auth, SHA pinning, concurrency groups, environment protection, branch protection |

## Gap Analysis

Current coverage is strong across core workflow mechanics, security, and infrastructure. Potential future additions:

| Gap | Priority | Notes |
|-----|----------|-------|
| Migration from other CI systems | Medium | Jenkins-to-Actions, Azure Pipelines-to-Actions migration patterns |
| GitHub Actions for monorepos | Medium | Path filtering, conditional job execution, Nx/Turborepo integration |
| Cost optimization | Low | Billable minutes tracking, runner sizing, workflow efficiency metrics |
| GitHub Packages integration | Low | Publishing npm/NuGet/Docker to GitHub Packages from workflows |
| Advanced debugging | Low | `ACTIONS_STEP_DEBUG`, `ACTIONS_RUNNER_DEBUG`, act for local testing |
