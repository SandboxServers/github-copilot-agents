# Skills

Skills are reusable playbooks, checklists, decision frameworks, and style guides used across multiple agents. Each skill is **self-contained, discoverable, and referenced** by agents and agents' knowledge files when relevant to a task.

## What is a Skill?

A skill captures a pattern that:
- Is used by **3+ agents** OR saves significant duplication
- Is **independent of any single domain** (not specific to one agent's knowledge base)
- Provides **actionable guidance** for a task or decision
- Has **clear applicability rules** ("when to use")

**Examples:**
- `engagement-coordination-protocol` — Multi-agent handoff structure (used by all orchestrators)
- `terraform-style-guide` — Naming, formatting, PR checklist (used by Terraform Author and reviewers)
- `agent-voice-guide` — Define personality for agents (used when creating/auditing agents)
- `security-review-framework` — Threat modeling checklist (used by security and architecture agents)

## Directory Structure

```
skills/
├── README.md                                    # This file
├── agent-consistency-audit.skill/SKILL.md
├── agent-voice-guide.skill/SKILL.md
├── api-design-standards.skill/SKILL.md
├── architecture-decision-records.skill/SKILL.md
├── bicep-style-guide.skill/SKILL.md
├── cost-optimization-checklist.skill/SKILL.md
├── database-migration-checklist.skill/SKILL.md
├── disaster-recovery-planning.skill/SKILL.md
├── documentation-standards.skill/SKILL.md
├── engagement-coordination-protocol.skill/SKILL.md
├── error-handling-patterns.skill/SKILL.md
├── iac-review.skill/SKILL.md
├── infrastructure-testing.skill/SKILL.md
├── kubernetes-security-hardening.skill/SKILL.md
├── naming-conventions.skill/SKILL.md
├── network-security-design.skill/SKILL.md
├── observability-baseline.skill/SKILL.md
├── pipeline-quality-gates.skill/SKILL.md
├── powershell-style-guide.skill/SKILL.md
├── secrets-management-audit.skill/SKILL.md
├── terraform-style-guide.skill/SKILL.md
└── testing-strategy.skill/SKILL.md
```

**Total: 23 skills**

## Skill Metadata

Each skill is a single `SKILL.md` file with YAML frontmatter:

```yaml
---
name: Display Name
description: One-line description of what this skill provides
when-to-use: When agents/users should reference this skill
categories:
  - Category1
  - Category2
---
```

## Skills by Category

### Agent Design & Quality
- `agent-voice-guide` — Personality and communication style for agents
- `agent-consistency-audit` — Resolve conflicts between agents on overlapping domains

### Multi-Agent Orchestration
- `engagement-coordination-protocol` — Mandatory pattern for multi-agent engagements, handoff templates, AGENT-CALLS.json structure

### Architecture & Design Patterns
- `api-design-standards` — REST conventions, versioning, error responses
- `architecture-decision-records` — ADR template and process
- `disaster-recovery-planning` — DR patterns, RTO/RPO tradeoffs
- `error-handling-patterns` — Retry, circuit breaker, saga, idempotency
- `observability-baseline` — Logging, metrics, tracing, alerting standards

### Code Style & Standards
- `bicep-style-guide` — Naming, file layout, decorators, PR criteria
- `naming-conventions` — Azure resources, Terraform, Bicep, PowerShell naming
- `powershell-style-guide` — Naming, splatting, error handling, templates
- `terraform-style-guide` — Naming, formatting, file organization, PR checklist

### Code Review & Quality Gates
- `iac-review` — Infrastructure code review checklist (Terraform, Bicep, K8s, CloudFormation)
- `infrastructure-testing` — Testing pyramid, CI pipeline integration
- `pipeline-quality-gates` — Azure Pipelines and GitHub Actions quality gate configuration
- `testing-strategy` — Test pyramid, quality gates, shift-left

### Security
- `kubernetes-security-hardening` — AKS cluster, pod, network, image security
- `network-security-design` — Private endpoints, NSGs, zero trust, WAF
- `secrets-management-audit` — Key Vault config, rotation, detection
- `security-review-framework` — Threat modeling and security review checklist

### Operations & Compliance
- `cost-optimization-checklist` — Waste elimination, right-sizing, commitments
- `database-migration-checklist` — SQL, PostgreSQL, MySQL, Cosmos migration workflows
- `documentation-standards` — README/runbook templates, Diátaxis framework

## How to Use a Skill

### As an Agent

Reference a skill in your agent's `## Related Skills` section:

```markdown
## Related Skills

- `terraform-style-guide` — Reference for naming, formatting, PR standards
- `infrastructure-testing` — For test pyramid and CI/CD integration guidance
```

When you need guidance on the skill's topic, **read the full SKILL.md file** to get templates, checklists, or decision frameworks.

### As a Knowledge File

Link to a skill from an agent's knowledge file (`agent-memory/[agent-name]/*.md`):

```markdown
## Naming Standards

Follow the naming conventions in the `naming-conventions` skill:
[Naming Conventions](../../skills/naming-conventions.skill/SKILL.md)
```

### As a User Prompt

When asking an agent for help with something a skill covers, mention it:

```
@azure-terraform-author Please write a module for App Service, following the terraform-style-guide skill.
```

## How to Add a New Skill

### 1. Check If It's Needed

A new skill is justified if:
- It's used by **3+ agents** OR saves significant duplication
- It's **not domain-specific** to one agent's knowledge base
- It captures a **reusable pattern or decision framework**

**Don't create a skill if:**
- The pattern is specific to one agent (use that agent's knowledge base instead)
- It's a one-off template or checklist (embed it in an agent or knowledge file)

### 2. Create the Skill File

Create `skills/{skill-name}.skill/SKILL.md`:

```yaml
---
name: Skill Display Name
description: One-line description
when-to-use: When agents/users should reference this skill
categories:
  - Category
  - Category2
---

# Skill Display Name

[Content: templates, checklists, decision frameworks, examples]

## Related Skills

- `other-skill-name` — When you need this after/before

## Reference

[Links to agent knowledge files that reference this skill]
```

### 3. Update Documentation

Update these files:

- **CLAUDE.md** — Add skill to the "Current Skills (N)" table and increment count
- **skills/README.md** — Add skill to directory structure and category section
- **Any agent files that reference it** — Add to `## Related Skills` sections
- **Any knowledge files that reference it** — Add links from relevant topics

### 4. Test & Commit

```bash
git add skills/{skill-name}.skill/SKILL.md CLAUDE.md skills/README.md
git commit -m "feat: add {skill-name} skill

[Description of what the skill does and why it was added]

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
git push origin main
```

## Maintaining Skills

### Update an Existing Skill

1. Edit the relevant `SKILL.md` file
2. Update the "Last Updated" date if the skill has one
3. If scope changed significantly, update CLAUDE.md description
4. Commit with a clear message on what changed and why

### Deprecate a Skill

If a skill is no longer needed:

1. Add a deprecation notice at the top of the SKILL.md file
2. Reference the replacement skill if applicable
3. Update CLAUDE.md to mark it deprecated
4. Remove references from agent files
5. Commit and communicate the deprecation

## Skill Inventory Summary

| Skill | Category | Agents Using |
|-------|----------|--------------|
| **engagement-coordination-protocol** | Multi-agent | All orchestrators |
| `agent-voice-guide` | Agent design | Used when adding/auditing agents |
| `agent-consistency-audit` | Quality | Used when finding agent conflicts |
| `terraform-style-guide` | Code style | Terraform Author, reviewers |
| `bicep-style-guide` | Code style | Bicep Author, reviewers |
| `naming-conventions` | Standards | All code authors |
| `cost-optimization-checklist` | Operations | Cost Specialist, leads |
| `security-review-framework` | Security | Security Analyst, leads |
| `testing-strategy` | Quality | Testing Engineer, code authors |
| `iac-review` | Code review | All code reviewers |
| [18 more skills...] | [Various] | [See CLAUDE.md] |

See `CLAUDE.md` for the complete skills table with descriptions.

## Cross-Skill References

- `agent-voice-guide` ↔ `agent-consistency-audit` — Used together to design coherent agents
- `terraform-style-guide` ↔ `naming-conventions` — Naming is part of style guidance
- `infrastructure-testing` ↔ `iac-review` — Testing requirements inform code review
- `security-review-framework` ↔ `network-security-design`, `secrets-management-audit`, `kubernetes-security-hardening` — Detailed guidance for security domains

## Questions?

- **How do I use a skill as an agent?** See "How to Use a Skill" section above
- **When should I add a new skill?** See "How to Add a New Skill" section
- **Where do I find a specific skill?** Use the category index or search by name in this directory
- **Can a skill reference another skill?** Yes — use cross-references in the "Related Skills" section

See `CLAUDE.md` for organizational context and agent-skill relationships.
