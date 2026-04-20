# CLAUDE.md — GitHub Copilot Agents Repository

This file documents the repository structure and conventions for Claude Code when working with this codebase.

## Project Overview

A collection of **31 custom agents** organized as a virtual Microsoft Cloud Engineering organization. Each agent is a domain specialist (or orchestrator lead) with:

- **Curated domain knowledge** stored in chunked markdown files (not monolithic) under `agent-memory/`
- **Specific tool access** — Azure APIs, GitHub, GitKraken, Microsoft Learn documentation
- **Defined relationships** — agents delegate to sub-agents, creating multi-agent workflows
- **Model tiering** — Opus 4.6 for orchestrators and code authors; Sonnet 4.6 for domain SMEs

**Key design principle**: Agents are built for both GitHub Copilot (VS Code) AND Claude Code, enabling drop-in install for either platform. The chunked knowledge system (agent-memories) ensures agents load only the context they need. The skill system (engagement-coordination-protocol, style guides, checklists) captures reusable patterns that scale across all agents and engagements.

**Engagement coordination**: When agents are called by `cloud-engineering-org` or act as leads in multi-agent orchestrations, they use the `engagement-coordination-protocol` skill to structure handoffs, track decisions, and enable retrospective analysis. This prevents context bloat in nested delegations and creates an audit trail for learning.

## Repository Structure

```
github-copilot-agents/
├── CLAUDE.md                    # This file — Claude Code guidance
├── README.md                    # User-facing overview and architecture diagram
├── agents/                      # All 31 agent definitions
│   ├── cloud-engineering-org.agent.md
│   ├── platform-engineering-lead.agent.md
│   ├── azure-ai-specialist.agent.md
│   └── [28 more agents...]
├── agent-memory/              # Chunked domain knowledge for each agent
│   ├── _copilot/                # Shared Copilot config (copilot-instructions.md)
│   ├── azure-ai-specialist/
│   │   ├── _toc.md              # Knowledge index — agents read this first
│   │   ├── ai-service-selection.md
│   │   ├── model-selection.md
│   │   └── [other topic chunks...]
│   ├── azure-compute-engineer/
│   └── [27 more agent memory folders...]
├── skills/                      # Reusable cross-agent playbooks and checklists
│   └── {skill-name}.skill/SKILL.md
├── prompts/                     # System prompts and shared instruction templates
└── .github/
    └── copilot-instructions.md  # VS Code Copilot workspace instructions
```

## Agent Structure

Each agent is a markdown file with YAML frontmatter:

```yaml
---
description: "..."        # What this agent does (1-2 sentences)
name: "..."              # Display name in Copilot Chat
model: "..."             # Claude Sonnet 4.6 or Opus 4.6
tools:                   # Array of MCP servers and capabilities
  - read
  - search
  - agent
  - com.microsoft/azure/*
  - microsoftdocs/mcp/*
agents:                  # Sub-agents this agent can delegate to
  - Agent Display Name
argument-hint: "..."     # Guidance for what users should tell this agent
handoffs:                # (Optional) Suggested delegations with custom prompts
  - label: "..."
    agent: agent-name
    prompt: "..."
---
```

**Content structure**:
1. **Core Principles** — Decision-making rules and non-negotiables
2. **Deep Knowledge Reference** — Points to `agent-memory/[agent-name]/_toc.md`
3. **Approach** — Step-by-step methodology for the agent's domain
4. **Constraints** — What the agent must NOT do
5. **Output Format** — What successful work looks like

## Knowledge Architecture (agent-memory/)

Knowledge is **chunked, not monolithic**. Each agent has a folder with:

- **`_toc.md`** — Knowledge index. Agents read this first, then load only relevant chunks.
- **Topic files** — Focused markdown files covering specific topics (e.g., `model-selection.md`, `cost-estimation.md`).

**Why chunking matters:**
- Agents load only context needed for the current task (saves tokens, faster responses)
- Compatible with Claude Code's memory system — chunks can be loaded as agent-specific memories
- Easier to maintain — updates to one topic don't affect others
- Better for multi-agent scenarios — sub-agents can reference parent knowledge without duplication

**Pattern in agents**:
```markdown
## Deep Knowledge Reference

**IMPORTANT**: Do NOT load all knowledge files at once. Follow this process:
1. First, read the knowledge index: [Knowledge Index](./agent-memory/[agent-name]/_toc.md)
2. Identify relevant topics
3. Load ONLY the relevant chunk files

This approach keeps context focused and avoids wasting tokens on irrelevant material.
```

## How to Add a New Agent

1. **Create the agent file** — `agents/new-agent-name.agent.md`
   - Use the structure above as a template
   - Be specific about what the agent does and doesn't do
   - List sub-agents if it's an orchestrator
   - **If orchestrator**: Add "Skills You MUST Use" section referencing `engagement-coordination-protocol`
   - Include tool requirements

2. **Create knowledge folder** — `agent-memory/new-agent-name/`
   - Create `_toc.md` with a knowledge index table (see any existing `_toc.md` for format)
   - Add topic files as needed (start with 3-5 critical topics)
   - Each file should be self-contained and cover one topic deeply

3. **Reference knowledge in the agent** — Add a "Deep Knowledge Reference" section pointing to the `_toc.md`:
   ```markdown
   [Knowledge Index](./agent-memory/new-agent-name/_toc.md)
   ```

4. **Update README.md** — Add the agent to the appropriate section (leads, specialists, shared services) in the inventory table.

5. **Link sub-agents** — If the agent is an orchestrator, add the sub-agents to:
   - The `agents:` array in YAML frontmatter
   - Add to "Skills You MUST Use" section: `engagement-coordination-protocol` (mandatory)
   - The parent agent's delegation section (if relevant)

## How to Update Knowledge

When updating chunked knowledge files:

1. **Update the relevant topic file** — Keep changes focused to the specific topic
2. **Update the `_toc.md`** — Modify the "When to Load" description if scope changed
3. **Update `_toc.md` timestamp** — Change the "Last Updated" date at the top
4. **Don't duplicate** — If information spans multiple agents, link between them rather than duplicating

Example update:
```markdown
> **Last Updated:** 2026-04-20 — Added PTU pricing tier guidance, updated model comparison table.
```

## How to Update References (After Folder Rename)

If `references/` is renamed to `agent-memory/`:

1. **Find all references** in agent files:
   ```bash
   grep -r "references/" agents/ prompts/
   ```

2. **Replace in agent files**:
   ```bash
   sed -i 's|./references/|./agent-memory/|g' agents/*.agent.md
   ```

3. **Verify the changes**:
   ```bash
   grep -r "agent-memory/" agents/ | head -5
   ```

4. **Commit the rename**:
   ```bash
   git mv references/ agent-memory/
   git add agents/
   git commit -m "refactor: rename references/ to agent-memory/ for clarity

   - Enables seamless integration with Claude Code memory system
   - Reflects the chunked knowledge design and memory-aware approach
   - Simplifies drop-in install for both GitHub Copilot and Claude Code"
   ```

## Model Tiering

- **Opus 4.6** — Orchestrators (leads) and code authors who make architectural decisions
  - `cloud-engineering-org`
  - `platform-engineering-lead`, `devops-lead`, `azure-specialty-lead`, `identity-productivity-lead`
  - `azure-terraform-author`, `bicep-code-author`
  - `ado-github-migration`
  - `azure-pipelines-architect`, `github-actions-architect`

- **Sonnet 4.6** — Domain specialists and shared services (cheaper, sufficient for expert Q&A)
  - All Azure service specialists (AI, Compute, Database, etc.)
  - Shared services (security, cost, testing, documentation, retrospectives)

## Delegation Patterns

Agents delegate in these ways:

1. **Orchestrator to specialists** — Lead (Opus) decomposes a request and routes pieces to specialists (Sonnet)
2. **Specialist to specialist** — Cross-domain expert consults another domain (e.g., Cloud Architect → Security Analyst)
3. **Any agent to code author** — Designer hands off for implementation (e.g., Architect → Terraform Author)
4. **Any agent to shared services** — Cost, security, testing, documentation consulted for reviews/analysis

## MCP Integration

Agents have access to:
- **Microsoft Learn** — `microsoftdocs/mcp/*` for official Azure documentation
- **Azure APIs** — Direct service access for pricing, configuration, monitoring
- **GitHub** — Repo search, issue/PR operations, code browsing
- **GitKraken** — Git history and operations

These are specified in the `tools:` array of each agent's YAML frontmatter.

## Skills

Skills are reusable playbooks, checklists, and decision trees used by multiple agents. Each skill lives in `skills/{skill-name}.skill/SKILL.md` with YAML frontmatter (`name`, `description`, `when-to-use`, `categories`).

### Current Skills (21)

| Skill | Description | Type |
|---|---|---|
| **engagement-coordination-protocol** | **Multi-agent handoff structure, AGENT-CALLS.json audit log, three handoff templates, retrospective integration** | **Mandatory for orchestrators** |
| `pipeline-quality-gates` | Azure Pipelines and GitHub Actions quality gate configuration | Shared infrastructure |
| `security-review-framework` | Threat modeling and security review checklist | Shared gate |
| `iac-review` | Infrastructure code review checklist (Terraform, Bicep, K8s, CloudFormation) | Code review |
| `architecture-decision-records` | ADR template and process | Documentation |
| `observability-baseline` | Logging, metrics, tracing, and alerting standards | Architecture |
| `terraform-style-guide` | Terraform naming, formatting, file organization, PR checklist | Code style |
| `bicep-style-guide` | Bicep naming, file layout, decorators, PR rejection criteria | Code style |
| `powershell-style-guide` | PowerShell naming, splatting, error handling, function template | Code style |
| `testing-strategy` | Test pyramid, quality gates, shift-left, anti-patterns | Quality |
| `api-design-standards` | Contract-first workflow, REST conventions, versioning, error responses | Architecture |
| `error-handling-patterns` | Retry, circuit breaker, saga, dead-letter, idempotency | Architecture |
| `naming-conventions` | Azure resources, Terraform, Bicep, PowerShell naming standards | Standards |
| `documentation-standards` | README/runbook templates, Diátaxis framework, writing style | Documentation |
| `infrastructure-testing` | Terraform/Bicep testing pyramid, CI pipeline integration | Quality |
| `cost-optimization-checklist` | Waste elimination, right-sizing, commitment discounts | Review gate |
| `disaster-recovery-planning` | DR patterns with RTO/RPO tradeoffs, failover testing | Architecture |
| `network-security-design` | Private endpoints, NSGs, zero trust, WAF, common findings | Security |
| `secrets-management-audit` | Auth hierarchy, Key Vault config, rotation, detection | Security |
| `database-migration-checklist` | SQL/PostgreSQL/MySQL/Cosmos migration workflows | Migration |
| `kubernetes-security-hardening` | AKS cluster, pod, network, and image security | Security |

Skills are referenced in agents' `## Related Skills` sections and in `agent-memory/_toc.md` files.

**When to create a skill**: The pattern is used by 3+ agents OR saves significant duplication across the codebase.

**When NOT to create a skill**: The knowledge is domain-specific to one agent (that's what agent-memories are for).

## Multi-Agent Engagement Coordination

### The engagement-coordination-protocol Skill (MANDATORY for Orchestrators)

When the `cloud-engineering-org` or division leads orchestrate multi-agent engagements:

1. **Create engagement folder** with structure defined in skill
2. **File-based handoffs** — Each agent call gets fresh context via specific files (SCOPE.md, prior outputs)
3. **AGENT-CALLS.json audit log** — Track what each agent was asked, what they returned, what made it into the plan
4. **ARCHITECTURE-PLAN.md specification** — Synthesized after all assessments, becomes the contract with user
5. **Retrospective integration** — When user delivers implementation, Retrospective Agent analyzes spec vs actual

### Why This Pattern

- **No context bloat** — Each agent operates in fresh context, no nested conversation history chains
- **Audit trail** — AGENT-CALLS.json enables retrospective analysis and organizational learning
- **Specification clarity** — ARCHITECTURE-PLAN.md is the contract; final delivery measured against it
- **Scaling** — Works for small 2-agent handoffs or large org-wide engagements

### Reference

See `skills/engagement-coordination-protocol.skill/SKILL.md` for:
- Complete engagement folder structure
- AGENT-CALLS.json schema with all fields
- Three reusable handoff templates (Division Assessment, Specialist Implementation, Review & Approval)
- Key practices for orchestrators and responding agents

## File Naming Conventions

- **Agents** — `lower-case-name.agent.md` (matches agent name in YAML)
- **Knowledge files** — `lower-case-topic.md` (no spaces, hyphens for readability)
- **Knowledge index** — `_toc.md` (underscore prefix for visibility)
- **Shared configs** — `copilot-instructions.md` in `agent-memory/_copilot/`

## Git Workflow

- Work on feature branches (e.g., `feat/add-caf-evangelist`, `feat/expand-ai-knowledge`)
- Keep commits focused — one agent addition per commit, or one knowledge update per commit
- Update `_toc.md` timestamp when knowledge changes
- Include summary in commit message of what changed and why (e.g., "Added responsible AI guidelines per compliance audit")

## Testing an Agent Locally

For GitHub Copilot (VS Code):
1. Place the agent folder in workspace or `~/.copilot/agents/`
2. VS Code discovers and lists agents automatically
3. Reference in Copilot Chat as `@agent-name`

For Claude Code:
1. Load the agent definition via the Agent tool with explicit prompt
2. Knowledge is loaded on-demand as agent processes questions

## Common Maintenance Tasks

### Add a new Azure service specialist
1. Create `agents/azure-[service]-specialist.agent.md`
2. Create `agent-memory/azure-[service]-specialist/` with knowledge chunks
3. Add to Azure Specialty Lead's sub-agents list
4. Update README inventory

### Expand knowledge for an existing agent
1. Create new `.md` file in `agent-memory/[agent-name]/`
2. Add row to `_toc.md` with "When to Load" guidance
3. Update timestamp in `_toc.md`

### Refactor an agent's approach
1. Update the Approach section in the agent file
2. Update Deep Knowledge Reference if topics changed
3. Test that handoffs and sub-agents still make sense

### Create a cross-cutting skill
1. Create `skills/{skill-name}.skill/SKILL.md` with YAML frontmatter (`name`, `description`, `when-to-use`, `categories`)
2. Document which agents should use it and when
3. Add to `## Related Skills` in relevant agent `.agent.md` files
4. Add to `## Related Skills` in relevant `agent-memory/_toc.md` files
5. Update the skills table in this file
