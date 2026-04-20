# Audience Adaptation

The same information needs different packaging for different readers. An executive doesn't need CLI commands. An on-call engineer doesn't need business justification. Identify your audience before you write a single word, then tailor everything — depth, format, length, and vocabulary — to what that audience needs.

## Audience Matrix

| Audience | What They Need | Preferred Format | Length | Technical Depth |
|---|---|---|---|---|
| **Executive** | Business impact, cost, timeline, risk | One-pager, slide deck | 1-2 pages | Low — outcomes and decisions only |
| **Engineering** | Implementation details, trade-offs, constraints | Technical spec, ADR, design doc | 5-15 pages | High — full detail with code |
| **Operations** | Step-by-step procedures, troubleshooting | Runbooks, playbooks | 2-5 pages | Medium — commands and config |
| **New hire** | Context, onboarding path, getting started | Onboarding guide, wiki | 5-10 pages | Progressive — start simple, layer depth |
| **External user** | How to use the product or API | API docs, user guide, quick start | Varies | Medium — focus on usage patterns |

## Writing for Each Audience

### Executives
Lead with business impact — cost savings, timeline, risk reduction. Executives make decisions based on outcomes, not implementation details. Remove all jargon and technical specifics. Include a clear "so what?" section with your recommendation. End with the decision or action you need from them. One page maximum — link to supporting detail for those who want it.

### Engineering
Show your work. Engineers trust documentation that includes alternatives considered, trade-offs evaluated, and constraints acknowledged. Include code samples, configuration snippets, and exact commands. Link to ADRs for architectural decisions. Be precise about limitations — handwaving erodes trust. Include diagrams for any interaction involving more than two components.

### Operations
Number every step — ambiguity at 2am causes outages. Include expected output after every command so the reader knows if it worked. Provide rollback procedures for every change that modifies state. Link to monitoring dashboards and alert definitions. Write assuming the reader is tired, stressed, and has never touched this system before.

### New Hires
Assume nothing. Define every acronym, explain every piece of context. Use progressive disclosure — start with the simplest explanation and layer complexity. Include "who to ask" for each topic area. Provide a clear onboarding path with checkpoints so the reader knows their progress. Link to deeper docs rather than overwhelming them upfront.

### External Users
Focus on the tasks they want to accomplish, not your internal architecture. Provide working code examples for every endpoint or feature. Include authentication, rate limits, error codes, and troubleshooting. Quick start should work in under 5 minutes. Version your documentation alongside your API — nothing is worse than docs that don't match the version the user is running.

## Adaptation Rules

- Write for the least experienced person in your target audience
- Never write "as you know" — if they knew, they wouldn't be reading your doc
- Provide escape hatches to deeper detail via links without cluttering the main content
- Same information, different packaging — reuse content across formats, adapt the presentation
- Test with a real member of the target audience before publishing
- When in doubt about the audience, write for operations: clear, step-by-step, no assumptions

## Mixed Audience Documents

Sometimes a document must serve multiple audiences. When this is unavoidable:

- Use an executive summary at the top for leadership
- Put detailed technical content in clearly labeled sections
- Use collapsible sections or links to separate depth levels
- Never sacrifice clarity for one audience to satisfy another
- Consider whether two separate documents would serve everyone better
