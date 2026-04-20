# Writing Style Guide

## Core Principles

Clear over clever. Short over long. Active voice over passive. Present tense over future. Second person ("you") for guides and tutorials. Concrete over abstract. Every sentence should earn its place — if removing it doesn't change the meaning, remove it.

## Sentence Construction

- Maximum 25 words per sentence — if it's longer, split it
- One idea per sentence — compound ideas get compound sentences
- Lead with the action: "Run the following command" not "The following command should be run"
- Put conditions first: "If you're using Premium tier, configure..." not "Configure... if Premium"
- Delete qualifiers that add no meaning: "basically", "actually", "simply", "just", "really"
- If a sentence needs multiple commas, it needs to be multiple sentences

## Paragraphs

- Maximum 4 sentences per paragraph
- One topic per paragraph — if you shift topics, start a new paragraph
- Lead with the key point — don't build up to it
- If a paragraph needs more than 4 sentences, it's covering two topics

## Headers

- Descriptive, not clever — "Configure authentication" not "Lock it down"
- Use H2 (`##`) for major sections, H3 (`###`) for subsections
- Never skip levels — H2 to H4 without an H3 is structurally wrong
- Sentence case: "Configure the gateway" not "Configure The Gateway"
- Headers should work as a scannable table of contents on their own

## Lists

- Bullet lists for unordered items — when sequence doesn't matter
- Numbered lists for sequences — when order matters
- Keep items parallel in grammatical structure
- Start each item with the same part of speech (all verbs, all nouns)
- Never nest more than 2 levels deep — restructure if you need deeper nesting

## Code Blocks

- Always specify the language identifier (```bash, ```powershell, ```json)
- Keep examples minimal but complete — the reader should be able to copy and run them
- Test all code examples before publishing — broken examples destroy trust instantly
- Use realistic but simple values — not `foo` and `bar`
- Mark placeholders clearly: `<RESOURCE_GROUP>` not `myResourceGroup`
- Show expected output when it helps verify the step worked

## Tables

- Use tables for comparisons and structured data — not for layout
- Keep columns under 5 — wide tables are hard to read
- Right-align numbers, left-align text
- Include units in column headers, not in individual cells
- Sort rows in a logical order: alphabetical, severity, or frequency

## Jargon and Acronyms

- Define on first use: "Center of Excellence (CoE)"
- Link to a glossary for project-specific terms
- Don't assume the reader knows acronyms — even common ones
- Spell out once, then abbreviate for the rest of the document

## Links

- Use descriptive text: `[Configure RLS](./rls-guide.md)` not `[click here](./rls-guide.md)`
- Link to the most specific section, not just the page
- Check links regularly — broken links erode confidence fast
- External links should open in a new tab when the platform supports it

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| **Wall of text** | No structure, no headings, no lists | Break into sections with headers and lists |
| **"Click here" links** | Meaningless without surrounding context | Use descriptive link text |
| **Burying the key info** | Reader has to dig for the answer | Put the answer first, then explain |
| **Outdated examples** | Worse than no examples at all | Test examples on every release |
| **Missing prerequisites** | Reader fails at step 1 | Add a prerequisites checklist at the top |
| **"As you know..."** | If they knew, they wouldn't be reading | Delete the phrase, explain the concept |
| **Passive voice overuse** | "The resource is created" — by whom? | "This command creates the resource" |
| **Screenshot-heavy docs** | Screenshots go stale with every UI change | Prefer text steps with diagrams |
