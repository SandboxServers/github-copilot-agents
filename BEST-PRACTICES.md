# GitHub Copilot Agents — Best Practices & Implementation Strategy

Based on VS Code Copilot best practices, practical utility analysis, and cross-platform integration (Copilot + Claude Code), here's what actually matters for this agent framework.

## VS Code Copilot Best Practices (Distilled)

### ✅ Agent Design Principles

1. **Match agent to task scope** — Narrow purpose > broad generic agent
   - One agent per specialty, not one generic "Azure expert"
   - Agents delegate to peers; leads orchestrate specialists
   
2. **Design for context efficiency** — Chunked knowledge > monolithic reference
   - Agents load what they need, not everything they know
   - `_toc.md` pattern is exactly right — keeps first query fast
   
3. **Prompts are instructions, not documentation**
   - Core Principles section tells agent how to think
   - Approach section is step-by-step methodology
   - Constraints section is what agent must NOT do
   - Links to knowledge chunks let agents load on-demand
   
4. **Model tiering is a cost/capability tradeoff**
   - Opus 4.6 for orchestrators and code authors (decisions matter)
   - Sonnet 4.6 for domain specialists (expert Q&A is enough)
   - Our current tiering is correct

5. **Tools are permissions, not features**
   - Enable only what the agent needs
   - Disable dangerous tools for read-only agents (don't give terraform authors `bash -c rm -rf`)
   - This is where hooks come in — safety gates at tool level

### ❌ Anti-Patterns (Things to Avoid)

1. **Don't embed all knowledge in the agent prompt**
   - Makes the agent huge, wastes context on irrelevant chunks
   - (We already avoid this with agent-memory/)

2. **Don't create agents that are just wrappers around MCP servers**
   - Agent = value-add on top of tools, not just "call this API"
   - Our agents have methodology + knowledge, not just tools

3. **Don't use handoffs for "call another agent"**
   - Handoffs are for structured workflows (design → implement → review)
   - Not just "I'll ask the database specialist"
   - Our current handoffs look good

4. **Don't assume agents will stay in context across turns**
   - Make prompts complete and self-contained per-agent
   - (Our agents are structured this way)

---

## What Actually Works for This Org: Practical Recommendations

### Phase 1: Foundation (Do This First) ⭐⭐⭐

**Status**: Ready to implement now. All pieces are in place.

1. **Agent Structure** (unified YAML for both platforms)
   ```yaml
   ---
   # Shared (both Copilot + Claude Code understand)
   name: "Azure Database Specialist"
   description: "[1-2 sentences: what, when to use]"
   tools: [read, search, web, agent, com.microsoft/azure/database, ...]
   
   # Copilot specific (Claude Code ignores)
   model: "Claude Sonnet 4.6 (copilot)"
   agents: [List of sub-agents]
   handoffs: [Structured workflows]
   argument-hint: "[Input guidance]"
   
   # Claude Code specific (Copilot ignores)
   model: claude-sonnet-4-6
   memory: project
   permissionMode: ask
   disallowedTools: [Write, Edit, Bash]  # Read-only agents
   maxTurns: 20
   ---
   ```
   
   **Why**: Single source of truth, works in both platforms, VS Code ignores fields it doesn't understand.

2. **Agent-Memory Chunking** (keep what you have)
   - `agent-memory/[agent-name]/_toc.md` — indexed by topic
   - Agents read TOC first, load only relevant chunks
   - **This is already right. Don't change it.**

3. **Skills** (use properly, not as separate agents)
   - Skills are `SKILL.md` folders, auto-discovered by description match
   - Copilot says: "I found the IaC Review skill, want me to apply it?"
   - **This is the right solution**, not creating more agents
   
   **Core 5 skills to build**:
   - `iac-review.skill/` — Checklist for Terraform/Bicep safety
   - `pipeline-quality-gates.skill/` — CI/CD gate standards
   - `architecture-decision-records.skill/` — ADR template + process
   - `security-review-framework.skill/` — Threat model + checklists
   - `observability-baseline.skill/` — Monitoring standards

4. **Documentation** (you have CLAUDE.md, that's it)
   - CLAUDE.md for Claude Code developers ✅
   - .github/copilot-instructions.md for Copilot users ✅
   - README.md for general overview ✅
   - That's all you need. Don't over-document.

---

### Phase 2: Safety Gates (Add After Phase 1 Works) ⭐

**Status**: Implement after skills are working. Measure the value first.

**Hooks** (shell commands at lifecycle events):
- Only use when you have a real safety problem
- Cost: adds latency, needs maintenance
- Only worth it if blocking dangerous operations

**Two hooks that make sense**:

1. **IaC Destruction Gate** (PreToolUse)
   ```json
   {
     "event": "PreToolUse",
     "tool": "bash",
     "pattern": "terraform destroy|az deployment delete",
     "action": "block",
     "message": "⚠️ Infrastructure destruction requires explicit approval",
     "requiresApproval": true
   }
   ```
   Prevents accidental `terraform destroy` without human say-so.

2. **Secret Scanning** (PreToolUse)
   ```json
   {
     "event": "PreToolUse",
     "pattern": "password|apikey|secret|token",
     "action": "block",
     "message": "⚠️ Potential secrets detected. Remove before committing."
   }
   ```
   Blocks agents from committing hardcoded credentials.

**Don't add more unless you see agents making the mistake repeatedly.**

---

### Phase 3: Enhance Knowledge (Ongoing) ⭐

**Status**: Already happening. Continue the pattern.

The merged PR added 11,000+ lines of anti-patterns, best-practices, gotchas across all agents. That's the right direction.

**Maintain this pattern**:
- Each agent knowledge folder has `_toc.md` + chunked topics
- Every `_toc.md` has a "Last Updated" timestamp
- Topics stay 3-8 KB (splits if larger)
- Update TOC when adding/removing topics

---

## What NOT to Do (Even Though It's Cool)

### ❌ Don't Create Agent Plugins

**Reason**: Plugins are for distributing agent bundles via marketplace. You're building internal agents.

Plugins add:
- Manifest complexity
- Version management overhead
- Marketplace registry setup

**When to revisit**: If you're distributing this to other teams/orgs.

### ❌ Don't Create Slash Commands (yet)

**Reason**: Agents are more powerful. Slash commands are for simple workflows.

Example anti-pattern:
- `/ask-database` slash command that asks the database agent
- Better: Just use @azure-database-specialist in chat

---

## Implementation Timeline

### Week 1: Merge Structural Changes
- [ ] Merge feat/knowledge-refinement to main (agents/, agent-memory/, CLAUDE.md)
- [ ] Update all agent files with unified YAML (both platform fields)
- [ ] Verify paths in all agent files point to agent-memory/

### Week 2: Build Core Skills
- [ ] IaC Review Skill (terraform-author, bicep-author, security-analyst)
- [ ] Pipeline Quality Gates Skill (devops-lead, both CI/CD architects)
- [ ] Test skill auto-discovery in Copilot Chat

### Week 3: Add Safety Gates (if needed)
- [ ] Implement IaC destruction hook (block terraform destroy without approval)
- [ ] Implement secret scanning hook
- [ ] Measure: do agents attempt dangerous operations?

### Week 4+: Enhance Knowledge
- [ ] Continue enriching `_toc.md` and topic chunks
- [ ] Build remaining skills (ADR, security-review, observability-baseline)
- [ ] Test in both Copilot and Claude Code

---

## File Structure (Final Recommended)

```
github-copilot-agents/
├── CLAUDE.md                           # Claude Code guidance ✅
├── BEST-PRACTICES.md                   # This file
├── README.md                           # User overview ✅
├── agents/                             # All 31 agents ✅
│   └── *.agent.md                      # Unified YAML + methodology
├── agent-memory/                       # Chunked knowledge ✅
│   ├── _copilot/
│   │   ├── conventions.md
│   │   ├── memory-guide.md
│   │   └── copilot-instructions.md
│   ├── _shared-skills/                 # (New) Shared knowledge chunks
│   │   ├── iac-safety/
│   │   ├── adrs/
│   │   ├── observability-standards/
│   │   └── security-baselines/
│   └── [agent-specific knowledge]
├── skills/                             # Skill.md folders ✅ (NEW)
│   ├── iac-review.skill/
│   │   ├── SKILL.md
│   │   ├── checklist.md
│   │   ├── examples/
│   │   └── decision-tree.md
│   ├── pipeline-quality-gates.skill/
│   ├── architecture-decision-records.skill/
│   ├── security-review-framework.skill/
│   └── observability-baseline.skill/
├── .github/
│   ├── hooks/                          # Safety gates (optional) ✅ (NEW)
│   │   ├── iac-safety.json
│   │   └── secret-scan.json
│   └── copilot-instructions.md         # Workspace config ✅
├── .claude/agents/                     # (Symlink to agents/ for Claude Code)
├── prompts/                            # Original persona prompts ✅
└── [other files...]
```

---

## Key Principles to Hold

1. **Agents are specialists with methodology, not API wrappers**
   - Core Principles + Approach + Constraints
   - Load knowledge on-demand via _toc.md

2. **Skills are discovered, not invoked**
   - Copilot matches description to task, auto-offers skill
   - Not: "I need to invoke the IaC review agent"
   - Instead: Copilot sees code, offers skill

3. **Hooks are safety gates, not quality checks**
   - Only block dangerous operations (terraform destroy, hardcoded secrets)
   - Don't use for linting or formatting (agents can do that)
   - Each hook adds latency — keep them minimal

4. **Context efficiency matters**
   - Agent prompt stays focused (not a knowledge dump)
   - Knowledge loaded on-demand (agents read _toc.md, load chunks)
   - Model tiering saves cost (Sonnet for specialists, Opus for leads)

5. **Documentation is minimal**
   - CLAUDE.md — development guidance
   - README.md — user overview
   - .github/copilot-instructions.md — workspace setup
   - In-code conventions — point to them, don't duplicate

---

## Success Metrics

After Phase 1, you'll know it's working when:
- [ ] Agents can be added to VS Code Copilot workspace and discovered automatically
- [ ] Agents work in Claude Code with memory system
- [ ] Agent files validate (YAML + referenced knowledge files exist)
- [ ] agents/ folder contains all 31 agents
- [ ] agent-memory/ contains chunked knowledge for 28 agents
- [ ] skills/ contains 5 core skills with proper SKILL.md format

After Phase 2:
- [ ] Copilot auto-discovers and offers IaC Review skill when user shares Terraform
- [ ] Safety hooks block dangerous operations without false positives

---

## Next Step

1. **Merge feat/knowledge-refinement to main** (structural refactor)
2. **Add best practices to every agent** (unified YAML with both platform fields)
3. **Create first skill** (iac-review.skill/ as proof of concept)
4. **Test in Copilot and Claude Code** before proceeding

Questions before proceeding?
