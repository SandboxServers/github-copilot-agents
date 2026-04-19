# Agent Prompts

This folder contains the original persona prompts used to create each agent in the organization. Each file is named after the corresponding `.agent.md` file and contains the raw prompt that defined the agent's personality, expertise, and behavioral boundaries.

## What These Are

These are the **seed prompts** — the initial descriptions that were used to generate each agent's system instructions. They capture the intended persona, domain expertise, and tone for each agent before those ideas were formalized into structured `.agent.md` frontmatter and knowledge files.

They serve as a reference for the *intent* behind each agent. If an agent's behavior drifts or its instructions get overly formal, the prompt file is the source of truth for what the agent was meant to be.

## Tone and Style

The prompts are written in a specific voice:

- **Lowercase, conversational, no corporate speak.** These read like a senior engineer describing a colleague, not a job posting.
- **Practical over theoretical.** Every claim is grounded in specific technologies, specific tradeoffs, specific gotchas — not vague assertions of competence.
- **Anti-pattern aware.** The prompts don't just say what the agent knows — they call out what goes wrong when people get it wrong. The $50k surprise bill. The policy push that breaks production. The service principal with god permissions no one remembers creating.
- **Opinionated but justified.** These agents have opinions (managed identity over SAS tokens, testing as a discipline not a checkbox, documentation as a product not an afterthought) and the prompts explain why.
- **Third person, present tense.** "this agent knows..." / "it handles..." / "it can..."
- **Em dashes for asides.** Parentheses for brief clarifications. No bullet points — each prompt is a single flowing paragraph.

## Writing New Prompts

When adding a new agent to the organization:

1. **Start with the prompt file.** Write the persona description first, before touching `.agent.md` frontmatter or knowledge files. The prompt is where you figure out what this agent actually *is*.

2. **Be specific.** Don't write "this agent knows Azure well." Write "this agent knows the azurerm provider deeply — the resource names, the argument patterns, the quirks, the bugs." Specificity is what makes these agents useful.

3. **Include the failure modes.** What goes wrong when this domain is handled poorly? What are the traps, the gotchas, the things that cost money or break production? An agent that only knows the happy path isn't useful.

4. **Define the boundaries.** What does this agent do vs. what does it delegate? "this agent writes terraform, not architectures" is a clear boundary. "this agent doesn't do the detailed work itself — it has specialists for that" is another.

5. **Name the relationships.** If this agent coordinates with or delegates to other agents, say so. The organizational structure should be visible in the prompts.

6. **One paragraph.** Each prompt is a single dense paragraph. No headers, no bullet points, no markdown formatting. Just a stream of expertise that reads like a person describing what they're good at.

7. **Match the filename.** The prompt file should be named `<agent-stem>.md` where `<agent-stem>` matches the agent's `.agent.md` filename without the extension (e.g., `azure-terraform-author.md` for `azure-terraform-author.agent.md`).

## File List

| File | Agent |
|------|-------|
| `azure-apps-infra-architect.md` | Azure Apps & Infra Architect |
| `azure-ai-specialist.md` | Azure AI Specialist |
| `azure-compute-engineer.md` | Azure Compute Engineer |
| `azure-data-engineer.md` | Azure Data Engineer |
| `azure-database-specialist.md` | Azure Database Specialist |
| `azure-integration-architect.md` | Azure Integration Architect |
| `azure-monitoring-engineer.md` | Azure Monitoring & Observability Engineer |
| `azure-network-engineer.md` | Azure Network Engineer |
| `azure-specialty-lead.md` | Azure Specialty Lead |
| `azure-storage-engineer.md` | Azure Storage Engineer |
| `azure-terraform-author.md` | Azure Terraform Author |
| `bicep-code-author.md` | Bicep Code Author |
| `caf-evangelist.md` | Cloud Adoption Framework Evangelist |
| `cloud-engineering-org.md` | Microsoft Cloud Engineering Organization |
| `cost-optimization-specialist.md` | Cost Optimization Specialist |
| `devops-lead.md` | DevOps Lead |
| `documentation-writer.md` | Documentation Writer |
| `entra-id-specialist.md` | Entra ID Specialist |
| `identity-productivity-lead.md` | Identity & Productivity Lead |
| `landing-zone-specialist.md` | Landing Zone Specialist |
| `m365-engineer.md` | Microsoft 365 Engineer |
| `platform-engineering-lead.md` | Platform Engineering Lead |
| `power-platform-engineer.md` | Power Platform Engineer |
| `powershell-automation-dev.md` | PowerShell Automation Developer |
| `retrospective-agent.md` | Retrospective Agent |
| `security-compliance-analyst.md` | Security & Compliance Analyst |
| `teams-admin.md` | Microsoft Teams Administrator |
| `testing-validation-engineer.md` | Testing & Validation Engineer |
