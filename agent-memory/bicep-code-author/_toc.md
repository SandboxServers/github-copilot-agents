# Bicep Code Author — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, AVM stance update, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Style Guide | [style-guide.md](style-guide.md) | **ALWAYS load first.** Naming, formatting, file layout, commenting, decorator conventions |
| Language Fundamentals | [language-fundamentals.md](language-fundamentals.md) | When working with types, UDTs, UDFs, decorators, loops, conditionals, lambdas |
| Import & Export | [import-export.md](import-export.md) | When sharing types, functions, or variables across files with `import`/`@export()` |
| Deployment Scopes | [deployment-scopes.md](deployment-scopes.md) | When deploying at resource group, subscription, management group, or tenant scope |
| Modules & AVM | [modules-avm.md](modules-avm.md) | When using local modules, registry modules, template specs, or studying AVM reference patterns |
| Parameter Files | [parameter-files.md](parameter-files.md) | When working with .bicepparam files, secure params, extending, environment configs |
| Deployment Stacks | [deployment-stacks.md](deployment-stacks.md) | When using deployment stacks — deny settings, actionOnUnmanage, CI/CD integration |
| What-If & Linter | [what-if-linter.md](what-if-linter.md) | When previewing changes, configuring linter rules in bicepconfig.json, validation pipeline |
| State & Deployment Model | [state-deployment-model.md](state-deployment-model.md) | When understanding ARM deployment model, incremental vs complete mode, existing resources |
| Best Practices | [best-practices.md](best-practices.md) | When reviewing code for security defaults, identity, private endpoints, diagnostics, project structure |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing code for common mistakes — hardcoding, missing conditions, secret outputs, monoliths |
| Gotchas | [gotchas.md](gotchas.md) | When debugging surprising behaviors — circular deps, what-if false positives, ARM limits, API versions |
| Deployment Troubleshooting | [deployment-troubleshooting.md](deployment-troubleshooting.md) | When diagnosing deployment failures — common errors, debugging commands, validation techniques |
