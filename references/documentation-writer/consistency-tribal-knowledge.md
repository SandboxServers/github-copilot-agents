## Consistency Patterns

### Terminology Management
1. Create a **glossary** for each project — define terms on first use, link to glossary
2. **Be consistent** — pick one term and stick with it (don't alternate between "environment" and "stage")
3. **Define acronyms** on first use: "Center of Excellence (CoE)" then "CoE" thereafter
4. **Don't assume** — what's obvious to the team that built it isn't obvious to the person reading at 2am

### Document Metadata
Every document should have:
- **Title** — clear, descriptive, searchable
- **Last updated** — date, not "recently"
- **Author/owner** — who to contact when it's wrong
- **Audience** — who this is for
- **Prerequisites** — what the reader needs before starting

### Review Checklist
Before publishing any document, verify:
- [ ] Could someone who wasn't in the room understand this?
- [ ] Are all acronyms defined on first use?
- [ ] Are all commands copy-pasteable?
- [ ] Are all links valid?
- [ ] Is the audience identified?
- [ ] Does it follow the right doc type (tutorial/how-to/reference/explanation)?
- [ ] Is there a "last updated" date?
- [ ] Has someone other than the author read it?

## Capturing Tribal Knowledge

### What to Document
1. **Why decisions were made** — not just what was decided (ADRs)
2. **Workarounds** — "we do X because Y has a bug/limitation" with link to issue
3. **Configuration rationale** — "this timeout is 30s because anything shorter causes Z"
4. **Gotchas** — "don't run this during peak hours because..."
5. **Who knows what** — team knowledge map (who's the expert on each system)
6. **Historical context** — "we migrated from A to B in Q3 2024 because..."

### Anti-Patterns
- **Wiki graveyards** — pages created once, never updated, never deleted
- **Documentation debt** — "we'll document it later" (you won't)
- **Screenshot-heavy docs** — screenshots go stale. Prefer text + diagrams.
- **Copy-paste from chat** — Slack/Teams conversations are not documentation
- **Docs without owners** — orphaned docs rot. Every doc needs an owner.
- **Perfect is the enemy of shipped** — an 80% accurate doc today beats a perfect doc never
