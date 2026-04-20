# CAF Evangelist — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| CAF Overview | [caf-overview.md](caf-overview.md) | When explaining the 7 CAF phases (Strategy through Manage) or how they relate |
| Strategy & Plan | [strategy-plan.md](strategy-plan.md) | When building business cases, defining motivations, planning digital estate assessment |
| Landing Zones | [landing-zones.md](landing-zones.md) | When explaining ALZ architecture, management group hierarchy, hub-spoke vs VWAN, subscription vending |
| Governance | [governance.md](governance.md) | When designing cloud governance — five disciplines, Azure Policy, maturity model |
| Migration Strategies | [migration-strategies.md](migration-strategies.md) | When discussing the 8 Rs of migration, Azure Migrate, or planning migration waves |
| Organizational Alignment | [organizational-alignment.md](organizational-alignment.md) | When advising on operating models, CCoE, skills gaps, RACI, or change management |
| CAF Anti-Patterns | [caf-anti-patterns.md](caf-anti-patterns.md) | When identifying CAF phase-specific mistakes in cloud adoption |
| Anti-Patterns (Broad) | [anti-patterns.md](anti-patterns.md) | When identifying broad organizational and strategic cloud adoption failures beyond CAF phases |
| Gotchas | [gotchas.md](gotchas.md) | When warning about non-obvious pitfalls — ALZ complexity, TCO gaps, policy enforcement timing |
| Best Practices | [best-practices.md](best-practices.md) | When recommending proven CAF adoption patterns — strategy-first, wave migration, governance from day 1 |

## Gap Analysis

The following topics have lighter coverage and could benefit from dedicated reference files:

| Gap | Current State | Recommendation |
|-----|--------------|----------------|
| Cloud Economics / Business Case | Covered briefly in strategy-plan.md and best-practices.md | Consider dedicated `cloud-economics.md` covering TCO methodology, Azure Pricing Calculator vs TCO Calculator, FinOps integration, reserved instance strategy, and Azure Hybrid Benefit qualification |
| Organizational Readiness Assessment | Covered in organizational-alignment.md at high level | Consider dedicated `readiness-assessment.md` covering skills gap analysis templates, change management frameworks, ADKAR model for cloud adoption, and training plan alignment to migration waves |
