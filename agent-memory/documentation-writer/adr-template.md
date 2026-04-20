# Architecture Decision Records (ADRs)

ADRs capture the "why" behind significant technical decisions. They are the institutional memory that survives team turnover. Without ADRs, future engineers reverse-engineer intent from code — or worse, undo good decisions because the reasoning was never written down.

## ADR Template

```markdown
# ADR-NNN: [Title — state the decision, not the problem]

## Status
[Proposed | Accepted | Superseded by ADR-XXX | Deprecated]

## Date
YYYY-MM-DD

## Deciders
[List of people involved in the decision]

## Context
What is the problem or situation? What forces are at play?
Include business context, technical constraints, timeline pressures,
and any relevant history.

## Decision
What did we decide? State it clearly and assertively.
Use present tense: "We use Cosmos DB for..." not "We will use..."

## Alternatives Considered

| Alternative | Pros | Cons | Why Rejected |
|---|---|---|---|
| Option A | ... | ... | ... |
| Option B | ... | ... | ... |

## Consequences

### Positive
- [What this decision enables or improves]

### Negative
- [What this decision constrains or costs]

### Risks
- [What could go wrong and how we mitigate it]

## Confidence Level
[High | Medium | Low] — Low confidence decisions should have a scheduled review date.

## References
Links to supporting documents, benchmarks, POC results, related ADRs.
```

## When to Write an ADR

- Choosing a technology, framework, or service (database, messaging, hosting)
- Changing architecture patterns (monolith to microservices, event sourcing)
- Establishing conventions that affect multiple teams or repos
- Making a decision that is expensive to reverse
- Rejecting an obvious option — document why the "obvious" choice was wrong

## ADR Rules

- **Immutable** — never edit an accepted ADR. Write a new one that supersedes it
- **Short** — one to two pages maximum. Link to design docs for full detail
- **Numbered** — sequential numbering makes cross-referencing easy ("per ADR-042")
- **Stored consistently** — in the repo (`/docs/adrs/`), wiki, or both. Pick one convention and stick with it
- **Titled as decisions** — "Use Cosmos DB for session state" not "Database selection"
- **Include confidence level** — low-confidence decisions need a scheduled review date
- **Break multi-phase decisions apart** — one ADR per decision, not one per project
- **Date and attribute** — always include when and who. Context changes over time

## ADR Lifecycle

| Status | Meaning |
|---|---|
| **Proposed** | Under discussion, not yet agreed |
| **Accepted** | Agreed and in effect |
| **Superseded** | Replaced by a newer ADR (link to it) |
| **Deprecated** | No longer relevant, kept for history |

## Common Mistakes

- Writing ADRs after the fact with no alternatives section — capture decisions when they're being made
- ADRs that describe the problem but never state a clear decision
- Storing ADRs where nobody can find them — discoverability is everything
- Treating ADRs as design documents — keep them focused on the decision, not the implementation
