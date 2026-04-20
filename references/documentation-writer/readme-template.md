# README Template

The README is the front door to your project. It's the first thing someone reads and it determines whether they keep going or leave. A good README answers three questions in under 60 seconds: what is this, why should I care, and how do I start.

## Standard README Structure

```markdown
# Project Name

One-sentence description of what this project does and why it exists.

## Quick Start

The fastest path from clone to running. Three commands or fewer.

\`\`\`bash
git clone <repo-url>
cd <project>
<run-command>
\`\`\`

## Prerequisites

- Runtime/language version (e.g., Node.js 20+, .NET 8, Python 3.11+)
- Required tools (e.g., Docker, Terraform, Azure CLI)
- Access requirements (e.g., Azure subscription, API keys)

## Installation

Step-by-step setup beyond the quick start. Include dependency installation,
environment configuration, and any first-run setup.

## Usage

Common usage examples. Show the most frequent commands or operations.
Include expected output where helpful.

## Configuration

Environment variables, config files, or settings the user may need to change.
Use a table for environment variables:

| Variable | Required | Default | Description |
|---|---|---|---|
| `DATABASE_URL` | Yes | — | Connection string for the database |
| `LOG_LEVEL` | No | `info` | Logging verbosity |

## Architecture (optional)

High-level overview for contributors. Include a diagram if the system
has more than 3 components. Link to ADRs for design decisions.

## Contributing

How to contribute: branch strategy, PR process, code style, testing requirements.
Link to CONTRIBUTING.md if it exists.

## License

License type and link to LICENSE file.
```

## README Rules

- Keep it under 2 screens of scrolling — link to deeper docs for details
- Quick Start should work in under 5 minutes for a developer with prerequisites met
- Include badges for build status, test coverage, and latest version where available
- Update the README when you change setup steps — stale READMEs are the #1 onboarding blocker
- Don't duplicate detailed docs in the README — link to them instead
- Test the Quick Start on a clean machine or fresh clone periodically

## What Doesn't Belong in a README

- Full API reference — put in dedicated reference docs
- Runbooks and incident procedures — put in ops documentation
- Architecture Decision Records — put in a `/docs/adrs/` folder
- Meeting notes or team processes — put in wiki or SharePoint
- Long-form tutorials — put in a `/docs/tutorials/` folder

## README Review Checklist

- [ ] Can a new developer go from clone to running in under 5 minutes?
- [ ] Are all prerequisites listed with exact version requirements?
- [ ] Are all commands copy-pasteable without modification?
- [ ] Is the project description clear to someone outside the team?
- [ ] Are links to deeper documentation included and working?
- [ ] Is there a "last updated" date or is it obviously current?
