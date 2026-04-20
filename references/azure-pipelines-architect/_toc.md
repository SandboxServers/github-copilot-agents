# Azure Pipelines Architect — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Template Patterns | [template-patterns.md](template-patterns.md) | When designing stage/job/step/variable templates, parameter types, conditional insertion, cross-repo refs |
| Expressions Reference | [expressions-reference.md](expressions-reference.md) | When working with compile-time, runtime, or macro expressions — functions, type casting, gotchas |
| Triggers & Resources | [triggers-resources.md](triggers-resources.md) | When configuring CI/PR/scheduled/pipeline triggers or repository/container/webhook resources |
| Variables & Secrets | [variables-secrets.md](variables-secrets.md) | When managing variable scopes, variable groups, Key Vault integration, output variables, or predefined vars |
| Environments & Approvals | [environments-approvals.md](environments-approvals.md) | When designing deployment strategies (runOnce/rolling/canary), approval gates, or environment checks |
| Caching & Artifacts | [caching-artifacts.md](caching-artifacts.md) | When optimizing builds with caching or managing pipeline/build artifacts |
| Security Hardening | [security-hardening.md](security-hardening.md) | When hardening pipeline security — service connections, protected resources, decorators, fork builds |
| Agent Pools | [agent-pools.md](agent-pools.md) | When selecting agent pools, configuring self-hosted agents, VMSS scaling, or estimating parallel jobs |
| Pipeline Optimization | [pipeline-optimization.md](pipeline-optimization.md) | When improving pipeline performance — parallelism, conditional execution, caching, analytics |
| Troubleshooting | [troubleshooting.md](troubleshooting.md) | When debugging pipeline failures — YAML errors, trigger issues, variable expansion, expression context |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing pipelines for common mistakes — monolithic YAML, no templates, plain-text secrets, over-privileged connections |
| Gotchas | [gotchas.md](gotchas.md) | When debugging subtle behaviors — expression timing, trigger path filters, variable precedence, multi-repo checkout paths |
| Best Practices | [best-practices.md](best-practices.md) | When designing pipelines from scratch — template architecture, environment protection, caching, security hardening, monitoring |

## Gap Analysis

Existing files (template-patterns through troubleshooting) cover core YAML mechanics well. The three new files fill these gaps:

- **Anti-Patterns** — codifies what NOT to do; previously no negative-pattern reference existed
- **Gotchas** — consolidates subtle behaviors scattered across expressions-reference, triggers-resources, and variables-secrets into one quick-lookup file
- **Best Practices** — provides a holistic design checklist; previous files covered individual features but lacked an overarching "how to architect well" guide

### Remaining gaps for future consideration
- **Migration Guide** — classic-to-YAML migration patterns (partially covered in anti-patterns)
- **Multi-Repo / Monorepo Strategies** — advanced trigger and checkout patterns for large codebases
- **Pipeline-as-Code Governance** — branch policies for pipeline YAML, required reviewers, template enforcement at org level
- **Cost Optimization** — parallel job sizing, agent pool right-sizing, build minutes budgeting
