# Documentation Anti-Patterns

Common mistakes that make documentation ineffective, untrustworthy, or ignored.
Recognizing these patterns is the first step — each has a concrete remedy.

## Audience Anti-Patterns

### Writing for Yourself, Not the Reader
- Author assumes the reader shares their context, history, and vocabulary
- Skips "obvious" background that isn't obvious to anyone outside the team
- **Fix:** Write for the least experienced person in your target audience. If you wrote the code, you're the worst judge of what's obvious
- Have someone outside the team read it before publishing — if they get stuck, the doc failed

### Too Much Jargon Without Definitions
- Acronyms and internal terms used without expansion or glossary
- Reader encounters "RBAC," "SPN," "CNAME" with no explanation
- **Fix:** Define every acronym on first use. Maintain a glossary for recurring terms
- Link to reference definitions rather than assuming familiarity

### Missing Prerequisites and Assumptions
- Guide starts with step 3 — steps 1 and 2 are "assumed knowledge"
- Reader fails silently because they don't have the right tooling, permissions, or context
- **Fix:** List prerequisites as a checklist at the top of every guide
- Include tool versions, required permissions, and environment setup

## Structure Anti-Patterns

### No Document Type Awareness
- Tutorial content mixed with reference tables mixed with architectural explanation
- Reader can't tell if they should read start-to-finish or look up a specific item
- **Fix:** Identify the document type (tutorial, how-to, reference, explanation) before writing
- Keep each document focused on a single type — link between types instead of mixing

### Walls of Text Without Structure
- Long paragraphs with no headings, no bullet points, no visual hierarchy
- Reader can't scan, can't find the section they need, gives up
- **Fix:** Use H2/H3 headings every 3–5 paragraphs. Break dense content into bullet lists
- Headers should work as a scannable table of contents on their own

### No Table of Contents for Long Documents
- Documents over 500 words with no way to navigate to specific sections
- Reader scrolls endlessly or gives up looking for the relevant part
- **Fix:** Add a table of contents for any document with 3+ sections
- Use auto-generated TOC features in your platform (GitHub, Docusaurus, MkDocs)

## Maintenance Anti-Patterns

### Outdated Documentation
- No review cadence, no "last updated" dates, no ownership
- Reader follows steps that worked 18 months ago — system has changed, doc hasn't
- **Fix:** Add "Last updated" and owner to every document. Review quarterly at minimum
- Treat stale docs as bugs — track them in your backlog, not in a wishlist

### Copy-Paste from Code Comments
- Code comments dumped into a doc without adapting for documentation context
- Inline comments make sense next to the code — they don't make sense on their own
- **Fix:** Rewrite code comments as standalone explanations with full context
- Documentation should explain _why_ and _when_, not just repeat _what_ the code does

### Documentation Lives Separate from Code
- Docs in a wiki nobody visits, disconnected from the repo where changes happen
- Code changes don't trigger doc updates — docs drift immediately
- **Fix:** Store docs in the same repo, same branch, same PR as the code they describe
- Docs-as-code: version control, pull request reviews, CI checks for broken links

## Content Anti-Patterns

### No Examples
- Abstract descriptions without concrete usage, commands, or sample output
- Reader understands the concept but can't translate it to their actual work
- **Fix:** Include at least one working example for every feature, API, or configuration
- Show input, command, and expected output — the reader should be able to copy and run it

### Over-Documenting Obvious Code, Under-Documenting Decisions
- Every getter and setter has a doc comment; no one documented why the retry policy is 5 attempts
- Trivial documentation crowds out the non-obvious reasoning that actually matters
- **Fix:** Document decisions, trade-offs, and constraints — not mechanics
- Use ADRs for architecture decisions, inline comments for non-obvious logic only

### Screenshots Without Alt Text or Captions
- Screenshots with no context — reader doesn't know what they're looking at or why
- Inaccessible to screen readers and useless when the UI changes
- **Fix:** Add descriptive alt text and a caption explaining what the screenshot shows
- Prefer annotated diagrams over screenshots — they survive UI redesigns better

## Sources

- [Microsoft Writing Style Guide](https://learn.microsoft.com/style-guide/welcome/)
- [Diátaxis documentation framework](https://diataxis.fr/)
- [Google developer documentation style guide](https://developers.google.com/style)
- [Documentation guidelines — MRTK2](https://learn.microsoft.com/windows/mixed-reality/mrtk-unity/mrtk2/contributing/documentation-guide)
