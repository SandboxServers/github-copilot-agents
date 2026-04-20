# GitHub Copilot Agents — Contributor Instructions

This repo contains 31 custom VS Code Copilot agents organized as a virtual Microsoft Cloud Engineering organization. Before making changes, understand the conventions below.

## Repo Structure

```
agents/             — 31 agent definitions (YAML frontmatter + markdown instructions)
prompts/            — Original persona prompts that seeded each agent
agent-memory/     — Chunked knowledge bases, one directory per agent
  <agent-name>/
    _toc.md         — Knowledge index (loaded first, points to relevant chunks)
    *.md            — Topic-specific knowledge chunks (loaded on demand)
README.md           — Repo overview, architecture diagram, agent inventory
```

## Quick Rules

- **Read before writing.** Before editing any agent file, read the full file and its `agent-memory/<agent-name>/_toc.md` to understand existing structure.
- **Agents are YAML frontmatter + markdown.** The frontmatter defines metadata (name, model, tools, agents, description). The markdown body is the system prompt.
- **Models are tiered.** Opus 4.6 for leads and code authors. Sonnet 4.6 for SMEs. See [conventions](agent-memory/_copilot/conventions.md) for the full mapping.
- **Agent references use display names.** The `agents:` list uses the target agent's `name:` field value, not the filename. Quote values containing `&` or `->`.
- **Knowledge is chunked.** Never put large reference material in the agent's `.agent.md` body. Put it in `agent-memory/<agent-name>/` with a `_toc.md` index.
- **Tools use slash format.** MCP tools are `com.microsoft/azure/<service>`, `microsoftdocs/mcp/*`, `github/mcp/*`, `gitkraken/*`. Core tools: `read`, `edit`, `search`, `web`, `execute`, `agent`, `todo`.

## Adding a New Agent

1. Write the persona prompt in `prompts/<agent-name>.md` first (see [prompts/README.md](prompts/README.md) for tone and style)
2. Create `agents/<agent-name>.agent.md` with YAML frontmatter following existing agents as templates
3. If the agent needs domain knowledge, create `agent-memory/<agent-name>/` with `_toc.md` and topic chunks
4. Add the agent to any parent agents' `agents:` lists using its `name:` value
5. Update [README.md](README.md) agent inventory tables

## Editing Existing Agents

- Don't change the `name:` field without updating every agent that references it in their `agents:` list
- Don't change model tier without justification — the tiering is intentional
- Keep knowledge chunks focused (3–8 KB each). Split if a chunk grows beyond that
- Update `_toc.md` when adding or removing knowledge chunks
- Preserve the "When to Load" guidance in `_toc.md` — it controls context efficiency

## Detailed Conventions

For model assignments, tool naming, MCP format, cross-reference rules, and knowledge architecture details, see the full conventions reference:

📄 [agent-memory/_copilot/conventions.md](agent-memory/_copilot/conventions.md)
