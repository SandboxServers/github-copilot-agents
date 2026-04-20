# Documentation Types — Diátaxis Framework

Every piece of documentation serves one of four purposes. Mixing them creates docs that fail at all four. Identify the type before you start writing, and keep each document focused on a single type.

## The Four Types

| Type | Purpose | Orientation | Analogy |
|---|---|---|---|
| **Tutorial** | Learning | Learning-oriented | Teaching a child to cook |
| **How-to Guide** | Task completion | Goal-oriented | Recipe in a cookbook |
| **Reference** | Information | Information-oriented | Encyclopedia article |
| **Explanation** | Understanding | Understanding-oriented | Article about culinary history |

## Tutorial

A tutorial is a guided learning experience that takes a beginner from zero to a working result. The reader doesn't yet know what questions to ask, so you lead them through every step. Tutorials build confidence — the reader should feel successful at the end. The measure of a good tutorial is that someone with no prior experience can follow it start to finish without getting stuck.

- State the outcome at the top: "By the end, you will have..."
- One clear path — no options, no alternatives, no branching
- Every step produces a visible result the reader can verify
- Concrete and reproducible — pin versions, use exact commands
- Test end-to-end before publishing — broken tutorials destroy trust
- Example: "Build your first Azure Function" not "Azure Functions overview"

## How-to Guide

A how-to guide solves a specific, real-world problem for a reader who already has competence. Unlike tutorials, how-to guides assume the reader knows the basics and just needs the steps. They are practical, focused, and task-oriented. Think of them as recipes — the reader picked this recipe because they want this specific outcome.

- Start with the goal, then give steps — minimum needed to succeed
- Include prerequisites as a checklist at the top
- Multiple approaches are acceptable when trade-offs differ
- One guide per task — don't combine unrelated tasks in one document
- Example: "How to deploy to AKS" not "Learn Kubernetes"
- Link to reference docs for parameter details instead of inlining them

## Reference

Reference documentation describes the machinery — accurate, complete, and consistent. It exists to be looked up, not read front to back. The reader knows what they're looking for and needs to find it fast. The quality of reference docs is measured by completeness and accuracy, not readability or narrative flow.

- API docs, config options, parameter lists, CLI flags
- No tutorial or how-to content — just facts and specifications
- Consistent structure across all entries — same headings, same order
- Auto-generate where possible from OpenAPI specs or code comments
- Update with every code change — stale reference is worse than none
- Dry and precise — personality is a distraction in reference material

## Explanation

Explanation documentation discusses the "why" behind decisions, designs, and trade-offs. It provides context that makes the other three types make sense. ADRs are a form of explanation. This is where you capture tribal knowledge — the reasons behind configurations, the history of migrations, the constraints that shaped the architecture.

- Answer "why" and "how does it work" — not "how to use it"
- Trade-offs, alternatives considered, historical context, constraints
- Can reference tutorials and how-to guides but don't duplicate them
- Stay factual and cite decisions — explanations are not opinion pieces
- This is where institutional knowledge lives and survives team turnover
- Example: "Why we chose Cosmos DB over PostgreSQL" not "How to use Cosmos DB"

## Choosing the Right Type

| Reader Says | They Need | Write a |
|---|---|---|
| "I want to learn X" | Guided experience | Tutorial |
| "I need to do X" | Practical steps | How-to Guide |
| "What does X do?" | Facts and specs | Reference |
| "Why does X work this way?" | Context and rationale | Explanation |

When in doubt, ask: "Is the reader learning, doing, looking up, or understanding?"

## Common Mistakes

- Mixing tutorial content into reference docs — the reader can't scan for facts
- Writing a how-to guide that's actually a tutorial — too much hand-holding for the audience
- Explanation docs that become opinion pieces — stay factual, cite decisions
- Reference docs that skip "boring" parameters — completeness is the entire point
- Tutorials with too many options ("you could also...") — pick one path and commit
- How-to guides without prerequisites — the reader fails at step 1 and blames the doc

## Cross-References Between Types

Good documentation links between types without duplicating content. A tutorial can link to reference docs for "all available options." A how-to guide can link to an explanation for "why we do it this way." Reference docs can link to tutorials for "getting started." Keep each type pure and use links to connect them.
