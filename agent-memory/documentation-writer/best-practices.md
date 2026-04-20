# Documentation Best Practices

Proven patterns for writing technical documentation that stays useful, accurate, and trusted.
Implement in order: structure first, then content quality, then process.

## Document Type Model (Diátaxis)

Apply the four-type model before writing. Every document serves one purpose — mixing types creates docs that fail at all four.

| Type | Purpose | Key Principle |
|---|---|---|
| **Tutorial** | Learning | Lead the reader through every step to a working result. No branching, no options |
| **How-to Guide** | Task completion | Solve one specific problem. Prerequisites up front, steps in order |
| **Reference** | Information lookup | Complete, accurate, consistent structure. Generated from code where possible |
| **Explanation** | Understanding context | Discuss why, not how. Capture trade-offs, constraints, and history |

- Identify the type before you start writing — put it in the document's front matter or header
- Link between types instead of mixing: "For background on why this approach was chosen, see [ADR-042](./adr-042.md)"
- Tutorials and how-to guides go stale fastest — prioritize them for review cycles

## Architecture Decision Records (ADRs)

Capture the _why_ behind decisions so future teams don't reverse good choices or repeat bad ones.

- **Status:** Proposed → Accepted → Deprecated → Superseded
- **Context:** What problem or question triggered this decision? What constraints apply?
- **Decision:** What was decided and why this option over the alternatives?
- **Consequences:** What trade-offs result? What new constraints does this create?
- Store ADRs in the repo: `/docs/adr/` or `/docs/architecture-decisions/`
- Number sequentially: `adr-001-use-cosmos-db.md`, `adr-002-event-driven-architecture.md`
- Never delete ADRs — supersede them with a new ADR that references the old one

## README Template

Every repo needs a README that answers five questions in under 2 minutes of reading:

1. **What** — one-paragraph description of what this project does
2. **Why** — the problem it solves or the value it provides
3. **How** — quick start with exact commands to get running locally
4. **Prerequisites** — tools, versions, permissions, and environment setup
5. **Links** — deeper docs, contributing guide, API reference, support channels

- Maximum 2 screens of scrolling — link to deeper docs for everything else
- Include badges: build status, coverage, latest version, license
- Test the quick start on a clean machine quarterly — broken quick starts destroy trust

## Runbooks

Operational runbooks must be executable at 2am by someone who has never touched the system.

- **Number every step** — no ambiguity about sequence
- **Include exact commands** — not "then deploy the thing" but `az webapp deploy --name myapp --src-path ./dist`
- **Show expected output** — so the reader knows each step worked
- **Provide rollback** — every change that modifies state needs a reversal procedure
- **Link to dashboards** — monitoring, alerts, and logs for verification
- Test runbooks during game days — untested runbooks are fiction

## Docs-as-Code Process

### Version Docs with Code
- Documentation lives in the same repo, same branch, same PR as the code it describes
- Code changes without doc updates should fail code review — treat it as a missing test
- Tag documentation versions alongside releases for API and SDK docs

### Review Docs Like Code
- Documentation changes go through pull request review — same as code
- Reviewers check accuracy, clarity, completeness, and adherence to style guide
- At least one reviewer should be outside the authoring team (fresh eyes catch assumptions)

### Automated Quality Checks
- Lint markdown in CI: `markdownlint`, `vale`, or `textlint` for style enforcement
- Check links automatically: `markdown-link-check` or `lychee` on every PR
- Validate code examples compile and run — broken examples destroy trust instantly
- Spell check with a technical dictionary that includes your project's terms

## Writing Style Essentials

- **Active voice:** "Configure the gateway" not "The gateway should be configured"
- **Second person:** "You deploy the app" not "The user deploys the app"
- **Present tense:** "This command creates a resource group" not "This command will create..."
- **Concrete over abstract:** Show the command, the output, the configuration — not just the concept
- **One idea per sentence:** Maximum 25 words. If it needs a semicolon, it needs to be two sentences

## Templates for Consistency

- Create templates for each document type: README, ADR, runbook, how-to, API reference
- Store templates in the repo: `/docs/templates/` or a shared documentation standards repo
- Templates enforce consistent structure — section order, required fields, metadata
- Review and update templates quarterly as team practices evolve

## Sources

- [Diátaxis documentation framework](https://diataxis.fr/)
- [Microsoft Writing Style Guide](https://learn.microsoft.com/style-guide/welcome/)
- [Docs-as-code approach](https://www.writethedocs.org/guide/docs-as-code/)
- [ADR GitHub organization](https://adr.github.io/)
- [Google developer documentation style guide](https://developers.google.com/style)
