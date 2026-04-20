# Claude Code + Copilot Blended Best Practices

This document identifies Claude Code best practices that strengthen our agent framework, plus what to explicitly avoid because it's incompatible with multi-agent Copilot architecture.

---

## ✅ Claude Code Principles We Should Adopt

### 1. **Keep Agent Prompts Ruthlessly Focused**

Claude Code: *"If your CLAUDE.md is too long, Claude ignores half of it."*

**Applied to agents**: Agent prompt bodies should be focused on methodology, not knowledge dumps.

- ✅ **Include in agent prompt**:
  - Core Principles (how the agent thinks)
  - Approach (step-by-step methodology)
  - Constraints (what NOT to do)
  - Link to chunked knowledge: "Read [Knowledge Index](./agent-memory/[name]/_toc.md)"

- ❌ **Don't include**:
  - Full reference material (that goes in agent-memory/)
  - Exhaustive service documentation (link to it instead)
  - Detailed Azure API references (that's what MCP is for)

**Why**: Copilot agents have limited token budget. Every bloated prompt means less context for the actual task.

Example good agent prompt structure (keep under 2KB for core):
```markdown
# Azure Database Specialist

You are a senior database architect...

## Core Principles
1. Right-size everything
2. Prefer managed services over DIY
3. [etc - 5-10 principles max]

## Approach
1. Understand the workload
2. Select appropriate service
3. [etc - 5-7 steps]

## Constraints
- Don't write migration scripts
- Don't design backups
- [etc]

## Deep Knowledge Reference
[Knowledge Index](./agent-memory/azure-database-specialist/_toc.md)
```

### 2. **Provide Clear Verification Criteria**

Claude Code: *"Claude performs dramatically better when it can verify its own work... without clear success criteria, it might produce something that looks right but doesn't work."*

**Applied to code-writing agents** (terraform-author, bicep-author, testing-engineer):

Always include verification in the prompt:

```markdown
## What Success Looks Like

✅ Terraform code:
- Runs `terraform validate` with no errors
- Passes `terraform plan` for idempotent changes
- Includes all variables in `variables.tf`
- Has `terraform fmt` applied

When you're done, run:
1. terraform validate
2. terraform plan
3. Show me the plan output
```

Same for Bicep, PowerShell, GitHub Actions. Agents write better code when they know how to verify it.

### 3. **Separate Exploration from Planning**

Claude Code: *"Letting Claude jump straight to coding can produce code that solves the wrong problem. Use Plan Mode to separate exploration from execution."*

**Applied to orchestrator agents** (leads, cloud-engineering-org):

Leads should explore first, plan second, delegate third:

```markdown
## Approach

1. **Explore** — Ask clarifying questions, understand constraints
2. **Plan** — Create a decomposition plan with sub-agents and sequence
3. **Delegate** — Route to specialists with specific prompts

Never jump to "call the terraform author." First ask:
- What does the user actually need?
- What constraints exist?
- Which specialists are needed in what order?
```

### 4. **Context is Your Most Precious Resource**

Claude Code: *"Most best practices are based on one constraint: Claude's context window fills up fast."*

**Applied to agents**:
- Chunked knowledge with _toc.md ✅ (we have this)
- Sub-agents for bounded investigation ✅ (we have this in agent delegation)
- Don't monolithic prompt bodies ✅ (guidance above)
- Load knowledge on-demand ✅ (chunked approach)

The principle: agents load only what they need, when they need it.

### 5. **Tool Scoping Matters**

Claude Code: *"Enable only the tools the agent needs."* (via disallowedTools and permission scoping)

**Applied to our agents**:

```yaml
---
# Read-only specialists (advisors, architects, analysts)
disallowedTools: [Write, Edit, Bash]
tools: [read, search, web, agent]

# Code-writing agents (authors, engineers)
disallowedTools: [Bash]  # No arbitrary shell commands
tools: [read, write, edit, execute, search, agent]

# Orchestrators (leads)
tools: [read, search, web, agent]  # No direct code changes
```

This is already in our unified YAML design. Keep it tight.

### 6. **Scope Investigations Narrowly**

Claude Code: *"When Claude investigates without scoping it, Claude reads hundreds of files, filling the context. Use subagents for research so exploration doesn't consume your main context."*

**Applied to multi-agent coordination**:

When an agent needs to investigate, it should use sub-agents for bounded research:

```markdown
## When You Don't Know Something

Instead of reading the entire codebase:
1. Ask a sub-agent to investigate (e.g., "Azure Terraform Author, check our existing variable patterns")
2. Sub-agent explores in its own context
3. Sub-agent reports back findings
4. Use findings to make your decision

This keeps your context clean for higher-level decisions.
```

This is how our orchestrators should work.

### 7. **Give Agents Skills They Can Invoke**

Claude Code: *"Create SKILL.md files in .claude/skills/ to give Claude domain knowledge and reusable workflows."*

**Applied to our framework**:

We're building this exactly right:
- `skills/iac-review.skill/` — reusable checklist
- `skills/pipeline-quality-gates.skill/` — reusable standards
- Copilot auto-discovers and offers them

This is high-leverage. Continue the plan.

---

## ❌ Claude Code Features We Should NOT Try to Adopt

### 1. **Non-Interactive Mode (`claude -p "prompt"`)**

**Why incompatible**: 
- Claude Code feature only (not Copilot)
- Runs CLI command, gets result, exits
- Our agents are conversational, interactive

**Philosophy conflict**: We're building for human-in-the-loop orchestration, not batch automation. Agents need to ask clarifying questions, show work, iterate.

**What to do instead**: Build skill workflows that humans invoke in chat.

### 2. **Parallel Independent Sessions**

**Why incompatible**:
- Claude Code desktop feature (multiple independent windows)
- Each session has separate context
- Agents would need to share state across sessions

**Philosophy conflict**: Our agents coordinate via conversation and delegation, not parallel independent work. The orchestrator (lead) decomposes and routes; specialists don't work in isolation.

**What to do instead**: Sub-agents within same conversation for bounded tasks.

### 3. **Agent Teams with Shared State**

**Why incompatible**:
- Advanced Claude Code feature for coordinating multiple independent sessions
- Requires persistent session state management
- Works great for "Agent A working on task, Agent B working on task, both reporting to team lead"

**Philosophy conflict**: Copilot agents run in a single conversation. There's no shared state between independent sessions.

**What to do instead**: Leads orchestrate via conversation, sub-agent calls keep everything in same context.

### 4. **Sandboxing/Worktrees**

**Why incompatible**:
- Claude Code feature for OS-level isolation
- Doesn't exist in Copilot

**Philosophy conflict**: Not applicable; agents work in user's actual workspace.

### 5. **Code Intelligence Plugins**

**Why incompatible**:
- Claude Code feature (symbol navigation, error detection after edits)
- Copilot doesn't have this depth of IDE integration

**Philosophy conflict**: Work within Copilot's constraints. Use MCP servers for context instead.

---

## How to Update Our Framework

### Update CLAUDE.md (Repo-Level)

Apply the "keep it focused" principle:

```markdown
# CLAUDE.md — GitHub Copilot Agents Repository

## Project Overview

31 custom agents organized as a virtual cloud engineering organization...

## Key Principles for Contributors

1. **Agent prompts are methodology, not knowledge dumps**
   - Core Principles, Approach, Constraints
   - Knowledge goes in agent-memory/, referenced as links
   - Keep agent prompt under 2KB

2. **Chunked knowledge is loaded on-demand**
   - Each agent-memory folder has _toc.md (index)
   - Agents read TOC first, load only relevant chunks
   - Don't duplicate across agents; link instead

3. **Code-writing agents verify their work**
   - Terraform: terraform validate, terraform plan
   - Bicep: bicep build, validation
   - GitHub Actions: workflow validation
   - Always include "What Success Looks Like" section

4. **Orchestrators explore → plan → delegate**
   - Don't jump straight to delegation
   - Ask clarifying questions first
   - Create decomposition plan
   - Route to specialists with specific context

5. **Use sub-agents for bounded investigation**
   - When you don't know something, ask a sub-agent
   - Keep your own context clean for decisions
   - Sub-agent explores, reports back, context returned to main agent

## Adding a New Agent

[Rest of existing CLAUDE.md...]
```

**Keep it this tight.** Remove anything else.

### Update BEST-PRACTICES.md

Add a new section: **Claude Code Principles Applied to Copilot Agents**

```markdown
## Claude Code Best Practices We've Adopted

### Focused Agent Prompts (Avoid Monolithic References)

Claude Code principle: *"If your CLAUDE.md is too long, the rules get lost."*

Applied to agents: Agent prompts should be focused on methodology (Core Principles, Approach, Constraints). Knowledge belongs in agent-memory/, not in the agent prompt body.

✅ Agent prompt: 1-2 KB
- Core Principles (thinking patterns)
- Approach (step-by-step)
- Constraints (what NOT to do)
- Link to chunked knowledge

❌ Don't put in agent prompt:
- Full API documentation
- Exhaustive service guides
- 10KB+ reference material

### Clear Verification Criteria (For Code-Writing Agents)

Claude Code principle: *"Claude performs better when it can verify its work."*

Applied to code-writing agents (terraform-author, bicep-author, testing-engineer):

```markdown
## What Success Looks Like

✅ When you're done:
- Terraform: runs `terraform validate` with no errors
- Bicep: runs `bicep build` successfully
- Tests: runs and passes all tests
- Actions: validates workflow syntax

Always include verification steps in your approach.
```

### Exploration → Planning → Delegation (For Orchestrators)

Claude Code principle: *"Separate research and planning from implementation."*

Applied to orchestrator agents (leads, cloud-engineering-org):

1. Explore — ask clarifying questions, understand constraints
2. Plan — decompose problem, identify specialists needed
3. Delegate — route to specialists with specific context

Don't jump straight to calling sub-agents.

### Scope Investigations Narrowly (Using Sub-Agents)

Claude Code principle: *"Unscoped investigation pollutes context. Use sub-agents for research."*

Applied to multi-agent coordination: When an agent needs to investigate something it doesn't know:

1. Ask a sub-agent to research (bounded scope)
2. Sub-agent explores in its own context
3. Sub-agent reports findings
4. Main agent uses findings for decision

This keeps your context available for higher-level work.

### Tool Scoping (Permissions)

Claude Code principle: *"Enable only what the agent needs."*

Applied to our agents: We scope tools in YAML via disallowedTools:

- **Read-only agents** (architects, advisors): read, search, web, agent only
- **Code-writing agents**: read, write, edit, execute, search, agent
- **Orchestrators**: read, search, web, agent (no direct code changes)

This is enforced by unified YAML with disallowedTools field.

[Keep BEST-PRACTICES.md, add this section]
```

### Update Agents Individually (Lower Priority)

When updating individual agents, apply the principle:
- Keep prompt body focused on methodology
- Link to knowledge: "See [Knowledge Index](./agent-memory/[name]/_toc.md)"
- For code-writing agents, add "What Success Looks Like" section
- Scope tools appropriately

---

## What's Unchanged (Our Philosophy Holds)

| Principle | Status |
|-----------|--------|
| 31 agents as orchestrators + specialists | ✅ Keeps holding |
| Chunked knowledge in agent-memory/ | ✅ Keeps holding |
| Model tiering (Opus for leads, Sonnet for SMEs) | ✅ Keeps holding |
| Skills as reusable playbooks | ✅ Keeps holding |
| Sub-agents for delegation | ✅ Keeps holding |
| Safety hooks for dangerous operations | ✅ Keeps holding |
| Cross-platform (Copilot + Claude Code) | ✅ Keeps holding |

---

## Summary: What We're Adding

From Claude Code best practices, we strengthen our agents by:

1. **Keeping agent prompts focused** — methodology only, knowledge on-demand
2. **Adding verification criteria** — code-writing agents verify their work
3. **Exploration → Planning → Delegation pattern** — orchestrators don't jump to delegation
4. **Bounded investigation** — sub-agents for research to keep context clean
5. **Tool scoping** — agents have only permissions they need (already in design)

None of these conflict with our direction. They reinforce it.

**What we're NOT adopting**:
- Non-interactive mode (not Copilot)
- Parallel independent sessions (not applicable)
- Agent teams with shared state (not applicable)
- Sandboxing (not applicable)

Our architecture is fundamentally different from Claude Code's (conversation-based orchestration vs. session-based parallelism). We take Claude Code's context-management principles and apply them to agent coordination.

---

## Implementation: Immediate Actions

1. **Tighten CLAUDE.md** — Apply "only include what would cause failure without it"
2. **Add verification sections** to terraform-author, bicep-author, testing-engineer
3. **Add exploration guidance** to cloud-engineering-org, platform-engineering-lead, etc.
4. **Keep the rest as-is** — our chunked knowledge design already solves Claude Code's context problems
