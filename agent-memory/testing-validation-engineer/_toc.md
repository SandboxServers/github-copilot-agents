# Testing & Validation Engineer — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Test Strategy | [test-strategy.md](test-strategy.md) | When designing test pyramid, choosing test ratios, or shift-left principles |
| Infrastructure Testing | [infrastructure-testing.md](infrastructure-testing.md) | When testing Terraform, Bicep, policies, smoke tests, or canary deployments |
| Load Testing | [load-testing.md](load-testing.md) | When designing load/stress/soak tests, Azure Load Testing, or identifying bottlenecks |
| Quality Gates | [quality-gates.md](quality-gates.md) | When configuring pipeline gates — coverage, static analysis, security scanning, approvals |
| Test Data Management | [test-data-management.md](test-data-management.md) | When managing synthetic data, test isolation, fixtures, or data masking |
| API Testing | [api-testing.md](api-testing.md) | When testing APIs — contracts, schemas, auth, rate limiting, backward compatibility |
| Chaos Engineering | [chaos-engineering.md](chaos-engineering.md) | When designing chaos experiments, Azure Chaos Studio, or gameday exercises |
| Test Automation | [test-automation.md](test-automation.md) | When automating CI test integration, parallelization, flaky test management, or reporting |
| Compliance Validation | [compliance-validation.md](compliance-validation.md) | When validating policy compliance, drift detection, or regulatory auditing |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing test suites for common mistakes, flaky tests, or inverted pyramid issues |
| Gotchas | [gotchas.md](gotchas.md) | When hitting unexpected tool behavior — Terraform auth, Bicep testing, JMeter quirks, coverage traps |
| Best Practices | [best-practices.md](best-practices.md) | When establishing testing standards — naming, determinism, quality gates, shift-left security |

## Gap Analysis

Existing coverage is strong for core testing disciplines. Potential future additions:

| Gap | Priority | Notes |
|-----|----------|-------|
| **Contract Testing** | Medium | Consumer-driven contracts (Pact), OpenAPI validation — partially covered in api-testing.md and best-practices.md, could warrant dedicated file |
| **Mutation Testing** | Low | Stryker.NET, mutation score as quality metric — advanced technique, not yet mainstream in Azure/IaC context |
| **Visual Regression Testing** | Low | Playwright screenshot comparison, Chromatic — relevant for UI-heavy workloads |
| **Accessibility Testing** | Medium | axe-core, Lighthouse CI, WCAG compliance gates — growing regulatory requirement |
| **Database Migration Testing** | Medium | Schema migration rollback verification, data integrity checks — not covered in current files |
| **Disaster Recovery Testing** | Medium | DR drill automation, RTO/RPO validation — overlaps with chaos-engineering.md but deserves focused coverage |
