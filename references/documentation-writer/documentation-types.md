## The Four Documentation Types (Diátaxis Framework)

Every piece of documentation falls into one of four categories. Mixing them produces docs that are hard to follow and impossible to maintain.

| Type | Purpose | Audience State | Structure | Example |
|---|---|---|---|---|
| **Tutorial** | Learning-oriented | "I want to learn" | Step-by-step guided experience | "Build your first Azure Function" |
| **How-To Guide** | Task-oriented | "I need to do X" | Steps to accomplish a specific goal | "How to configure RLS in Power BI" |
| **Reference** | Information-oriented | "I need to look up Y" | Dry, accurate, complete | API docs, CLI flags, config schema |
| **Explanation** | Understanding-oriented | "I want to understand why" | Discursive, contextual | "Why we chose Cosmos DB over SQL" |

### Key Rules per Type

**Tutorials**
- Lead the reader through a complete experience from start to finish
- Every step must produce a visible result
- Don't explain choices — make them for the reader
- Assume nothing except stated prerequisites
- Test every step yourself before publishing
- State the outcome at the top: "By the end of this tutorial, you will have..."

**How-To Guides**
- Assume the reader already knows the basics
- Start with the goal, then give steps
- One guide per task — don't combine unrelated tasks
- Include prerequisites as a checklist, not a tutorial
- "If you need to..." not "Let's learn about..."

**Reference Documentation**
- Completeness over readability — every parameter, every option, every return value
- Consistent structure across all entries (same headings, same order)
- No narrative — just facts
- Keep examples minimal and focused on usage, not explanation
- Auto-generate where possible, hand-curate where necessary
- Update with every code change — stale reference docs are worse than none

**Explanations**
- Answer "why" and "how does it work" — not "how to use it"
- Provide context, history, tradeoffs, alternatives considered
- ADRs (Architecture Decision Records) are a form of explanation
- Can reference tutorials and how-to guides but don't duplicate them
- Where you capture tribal knowledge — "why is this configured this way?"
