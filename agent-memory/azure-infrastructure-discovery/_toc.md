# Azure Infrastructure Discovery — Knowledge Index

> **Last Updated:** 2026-04-20 — Initial knowledge base for discovery patterns, tooling, command references.

This agent discovers and assesses current Azure infrastructure state. It runs queries, reads configurations, and produces current-state baselines. Knowledge is minimal here because discovery is about observation, not domain expertise.

## Knowledge Chunks

| Topic | File | When to Load |
|-------|------|--------------|
| Discovery execution patterns | discovery-patterns.md | When planning what commands to run for different discovery scenarios |
| Azure CLI command reference | azure-cli-reference.md | When building discovery scripts for subscription, resource, policy, RBAC inventory |
| PowerShell patterns | powershell-patterns.md | When using PowerShell to query infrastructure state |
| KQL queries for assessment | kql-discovery-queries.md | When querying Log Analytics for compliance, audit, activity patterns |
| Compliance baseline assessment | compliance-assessment.md | When evaluating against SOC2, HIPAA, PCI-DSS, CAF baselines |
| Cost data extraction | cost-baseline-patterns.md | When building cost assessment reports and identifying cost drivers |
| Report structure | discovery-report-template.md | When organizing findings for handoff to org or specialists |

## Related Skills

- [security-review-framework](../../skills/security-review-framework.skill/SKILL.md) — Use for compliance assessment structure
- [cost-optimization-checklist](../../skills/cost-optimization-checklist.skill/SKILL.md) — Use for cost analysis
- [naming-conventions](../../skills/naming-conventions.skill/SKILL.md) — Use for assessing naming standard compliance
