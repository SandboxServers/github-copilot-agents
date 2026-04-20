# DevOps Lead — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| DevOps Practice | [devops-practice.md](devops-practice.md) | When explaining DevOps culture, CI/CD foundations, or shift-left principles |
| DORA Metrics | [dora-metrics.md](dora-metrics.md) | When measuring delivery performance — deployment frequency, lead time, CFR, MTTR |
| Pipeline Architecture | [pipeline-architecture.md](pipeline-architecture.md) | When designing template architecture, multi-stage pipelines, or pipeline security |
| ADO to GitHub Migration | [ado-to-github-migration.md](ado-to-github-migration.md) | When planning migration from Azure DevOps to GitHub |
| InnerSource | [innersource.md](innersource.md) | When sharing automation across teams — shared repos, contribution model |
| Pipeline at Scale | [pipeline-at-scale.md](pipeline-at-scale.md) | When managing 200+ pipelines — template versioning, agents, monitoring, cost |
| Maturity Assessment | [maturity-assessment.md](maturity-assessment.md) | When assessing DevOps maturity or prescribing improvements |
| DevOps Anti-Patterns (CI/CD) | [devops-anti-patterns.md](devops-anti-patterns.md) | When identifying CI/CD failure modes and manual process risks |
| Anti-Patterns (Broad) | [anti-patterns.md](anti-patterns.md) | When identifying organizational, process, and tooling failures across DevOps practice |
| Gotchas | [gotchas.md](gotchas.md) | When navigating subtle traps — DORA gaming, trigger interactions, slot stickiness |
| Best Practices | [best-practices.md](best-practices.md) | When prescribing delivery practices — shift left, immutable artifacts, progressive delivery |

---

### Gap Analysis

The following topics are adjacent to this agent's domain but are not yet covered
by dedicated reference files. Consider creating them if demand arises:

| Gap | Notes |
|-----|-------|
| InnerSource Strategy at Scale | Current `innersource.md` covers contribution models. Missing: governance for cross-org InnerSource, measuring InnerSource health, handling abandoned shared repos. |
| Platform Engineering Intersection | DevOps lead and platform engineering lead share significant overlap — self-service pipelines, golden paths, developer experience. A reference clarifying boundaries and collaboration points would reduce ambiguity. |
| Incident Management & SRE Practices | MTTR is a DORA metric but incident response runbooks, post-incident reviews, and SRE error budgets are not covered. |
| Cost Optimization for CI/CD | Pipeline compute costs, agent pool right-sizing, build caching strategies, and parallelism trade-offs are operational concerns without a dedicated reference. |
| Compliance as Code | Regulated industries need audit trails, change evidence, and policy enforcement. Current references touch security but not compliance automation specifically. |
