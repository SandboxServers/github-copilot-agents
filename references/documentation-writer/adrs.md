## Architecture Decision Records (ADRs)

### ADR Template

```markdown
# ADR-NNN: [Title — the decision, not the problem]

## Status
[Proposed | Accepted | Superseded by ADR-XXX | Deprecated]

## Context
What is the problem? What forces are at play? What constraints exist?
Include business context, technical constraints, and timeline pressures.

## Decision
What did we decide? State it clearly and assertively.

## Alternatives Considered
| Alternative | Pros | Cons | Why Rejected |
|---|---|---|---|
| Option A | ... | ... | ... |
| Option B | ... | ... | ... |

## Consequences
What are the positive and negative implications of this decision?
What new constraints does it introduce?

## Confidence Level
[High | Medium | Low] — Low confidence decisions should be revisited.

## References
Links to supporting documents, benchmarks, POC results.
```

### ADR Rules
- Append-only log — never edit accepted records, write new ones that supersede
- Include the confidence level — low-confidence decisions need scheduled review dates
- Keep records pithy and assertive — if you need a design guide, link to it separately
- Break multi-phase decisions into separate records per phase
- Store with the workload's documentation (repo, wiki, or both)
- Number sequentially — makes it easy to reference ("per ADR-042")
