# Documentation Writer — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Documentation Types | [documentation-types.md](documentation-types.md) | When choosing between tutorial, how-to, reference, or explanation formats |
| Writing Style | [writing-style.md](writing-style.md) | When applying writing style — voice, sentence construction, clarity |
| README Template | [readme-template.md](readme-template.md) | When writing or reviewing READMEs |
| ADR Template | [adr-template.md](adr-template.md) | When writing or reviewing Architecture Decision Records |
| Runbook Template | [runbook-template.md](runbook-template.md) | When writing incident runbooks or operational playbooks |
| Audience Adaptation | [audience-adaptation.md](audience-adaptation.md) | When adapting content for executives, engineers, ops, or new hires |
| Documentation Platforms | [documentation-platforms.md](documentation-platforms.md) | When choosing doc platforms or docs-as-code practices |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing docs for common mistakes — audience, structure, maintenance, content |
| Gotchas | [gotchas.md](gotchas.md) | When dealing with Markdown rendering, link issues, code blocks, platform quirks |
| Best Practices | [best-practices.md](best-practices.md) | When applying Diátaxis model, ADRs, docs-as-code, runbooks, templates |

## Gap Analysis

Current coverage is strong across documentation types, writing mechanics, templates, audience adaptation, and platform selection. The addition of anti-patterns, gotchas, and best-practices closes the primary gaps around what-to-avoid guidance, platform-specific pitfalls, and process-level practices.

**Remaining gaps for future consideration:**
- **Accessibility** — WCAG compliance for documentation, screen reader testing, color contrast in diagrams
- **Localization / i18n** — patterns for multi-language documentation, translation workflows
- **API documentation** — OpenAPI/Swagger generation, SDK doc patterns, versioning strategies
- **Metrics** — measuring documentation effectiveness (page views, time-on-page, support ticket deflection)
- **Migration playbooks** — moving docs between platforms (wiki → docs-as-code, Confluence → Docusaurus)

## Related Skills

| Skill | When to Load |
|-------|-------------|
| `documentation-standards` | README/runbook templates, Diátaxis framework, writing style rules |
| `architecture-decision-records` | ADR template and process |
