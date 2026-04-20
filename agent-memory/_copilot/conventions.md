# Repo Conventions — Detailed Reference

> Load this file when you need specifics on agent file format, model assignments, tool naming, or knowledge structure. The summary in `.github/copilot-instructions.md` covers the basics.

## Agent File Format

Agent files are `.agent.md` with YAML frontmatter. Required fields:

```yaml
---
description: "One-line description. Start with agent name. Include 'Use when:' triggers."
name: Display Name
tools:
  - read
  - search
  # ... tool list
agents:
  - Other Agent Display Name
  - "Agent Name With & Special Chars"
argument-hint: "What to tell the user to provide"
model: "Claude Sonnet 4.6 (copilot)"
---
```

**Field rules:**
- `description` — Always quoted. Starts with the agent's display name. Contains "Use when:" followed by comma-separated trigger scenarios.
- `name` — The display name. This is what other agents reference in their `agents:` lists.
- `model` — A single string, not a list. Exception: the 3 CI/CD agents use single-item list format (works fine).
- `tools` — Core tools first, then MCP tools. Never include tools the agent doesn't need.
- `agents` — Display names from target agents' `name:` fields. Double-quote any value containing `&`, `->`, or other YAML special characters. `Explore` is a built-in VS Code agent (not defined in this repo).
- `argument-hint` — Short guidance on what context the user should provide.

Optional fields:
- `handoffs` — Define suggested follow-up actions with label, target agent, and prompt.

## Model Tier Assignments

### Opus 4.6 (`"Claude Opus 4.6 (copilot)"`) — 11 agents
Division leads and code authors. Higher reasoning for orchestration, code generation, and migration planning.

| Agent | Reason |
|-------|--------|
| platform-engineering-lead | Division lead — orchestrates specialists |
| devops-lead | Division lead — coordinates CI/CD agents |
| azure-specialty-lead | Division lead — coordinates Azure SMEs |
| identity-productivity-lead | Division lead — coordinates M365/identity |
| azure-terraform-author | Code author — writes production Terraform |
| bicep-code-author | Code author — writes production Bicep |
| powershell-automation-dev | Code author — writes production PowerShell |
| ado-github-migration | Migration specialist — complex multi-step planning |
| azure-pipelines-architect | CI/CD architect — pipeline design and debugging |
| github-actions-architect | CI/CD architect — workflow design and debugging |
| retrospective-agent | Pattern analysis across engagements |

### Sonnet 4.6 (`"Claude Sonnet 4.6 (copilot)"`) — 20 agents
Domain SMEs and the orchestrator. Expert Q&A doesn't need the most expensive model.

All other agents including `cloud-engineering-org` (the top-level orchestrator uses Sonnet because it delegates immediately rather than doing deep work itself).

## Tool Naming

### Core tools (available to all agents)
| Tool | Purpose |
|------|---------|
| `read` | Read files |
| `edit` | Edit/create files |
| `search` | Search workspace |
| `web` | Web search |
| `execute` | Run terminal commands |
| `agent` | Call sub-agents (required if `agents:` is specified) |
| `todo` | Manage task lists |

### MCP tools (slash format)
| Pattern | Service |
|---------|---------|
| `com.microsoft/azure/<service>` | Azure service APIs (e.g., `com.microsoft/azure/compute`, `com.microsoft/azure/keyvault`) |
| `com.microsoft/azure/documentation` | Azure documentation |
| `microsoftdocs/mcp/*` | Microsoft Learn documentation search |
| `github/mcp/*` | GitHub operations |
| `gitkraken/*` | Git operations via GitKraken |

### CI/CD agent extra tools
The 3 CI/CD agents (`ado-github-migration`, `azure-pipelines-architect`, `github-actions-architect`) have additional tools:
- `vscode/askQuestions` — Prompt user for clarification
- `read/problems` — Read VS Code Problems panel
- `search/changes` — Search recent changes

## Knowledge Architecture

### Directory structure
```
agent-memory/<agent-name>/
  _toc.md           — Knowledge index, always loaded first
  topic-name.md     — Focused chunk on a single topic
```

### _toc.md format
```markdown
# Agent Name — Knowledge Index

> **Last Updated:** YYYY-MM-DD — [description of last update]

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Topic Name | [filename.md](filename.md) | When the user asks about X or Y |
```

### Chunk sizing
- Target 3–8 KB per chunk
- Each chunk should be useful in isolation
- Content: decision frameworks, comparison tables, configuration guidance, anti-patterns, probing questions
- Tone: practical, specific, opinionated where warranted

### Agents without knowledge directories
3 agents have no `agent-memory/` directory — their domain knowledge fits in the `.agent.md` prompt:
- `ado-github-migration`
- `azure-pipelines-architect`
- `github-actions-architect`

## Cross-Reference Rules

When one agent references another in its `agents:` property:
1. Use the target agent's `name:` field value (the display name), not the filename stem
2. Double-quote values with YAML special characters: `"Azure Pipelines -> GitHub Actions Migration Specialist"`
3. The `Explore` agent is a built-in VS Code agent — never change or remove it from agents lists
4. Some agent references have inline comments (e.g., `# Senior ADO pipeline specialist`) — preserve them

## File Naming

- Agent files: `agents/<kebab-case-name>.agent.md`
- Prompt files: `prompts/<kebab-case-name>.md` (matches agent filename without `.agent`)
- Knowledge directories: `agent-memory/<kebab-case-name>/`
- Knowledge chunks: `agent-memory/<kebab-case-name>/<kebab-case-topic>.md`
- TOC files: `agent-memory/<kebab-case-name>/_toc.md`

## Timestamps

All `_toc.md` files include a timestamp line:
```
> **Last Updated:** YYYY-MM-DD — [description of content source/changes]
```
Update this when modifying knowledge content.
