## Systemic Patterns

### Purpose
Individual retrospectives identify issues within a single engagement. Analyzing patterns
across multiple retrospectives reveals systemic problems that require organizational-level
solutions rather than team-level workarounds.

### What Makes a Pattern Systemic
A pattern is systemic when it:
- Appears across multiple engagements, teams, or time periods
- Persists despite individual teams attempting to address it
- Has root causes in shared processes, tooling, or organizational structure
- Cannot be fully resolved by a single team acting alone
- Requires changes to templates, standards, training, or platform capabilities

### Root Cause Categories
Classify recurring issues into categories to identify where systemic investment is needed:

#### Process
- Missing or unclear handoff procedures between teams
- Approval bottlenecks that consistently delay delivery
- Change control processes that don't scale with velocity
- Inconsistent definition of done across teams
- Missing review gates or reviews happening too late to influence design

#### Tooling
- Shared tooling with gaps that teams individually work around
- Pipeline templates that break with provider upgrades
- Monitoring and alerting configurations that miss common failure modes
- Cost management tooling not integrated into engineering workflows
- Security scanning tools with high false positive rates that get ignored

#### Skills
- Same knowledge gaps appearing across different teams
- Heavy reliance on specific individuals for certain expertise
- Onboarding process not preparing team members for common scenarios
- Training materials outdated or not reflecting current architecture patterns
- Skills matrix not maintained, making it hard to allocate expertise

#### Communication
- Information siloed within teams, not shared across the organization
- Decisions made without the right stakeholders involved
- Lessons learned documented but not discoverable or referenced
- Requirements ambiguity causing rework across engagements
- Stakeholder expectations misaligned with technical constraints

#### Architecture
- Recurring design issues that templates or reference architectures could prevent
- Infrastructure patterns that consistently cause operational problems
- Security configurations that are frequently misconfigured
- Networking designs that create recurring connectivity issues
- Monitoring gaps that delay incident detection

### Trend Analysis
Track patterns over time to measure improvement:
- Count occurrences of each pattern per quarter
- Measure severity: are instances getting less impactful even if frequency persists?
- Correlate with systemic interventions: did the new template reduce the finding rate?
- Identify emerging patterns before they become entrenched
- Report trends to leadership with data, not anecdotes

### Addressing Root Causes vs Symptoms
| Symptom Fix | Root Cause Fix |
|---|---|
| Add a manual review step | Automate the check in CI/CD |
| Document the workaround | Fix the underlying tooling gap |
| Train one team | Update onboarding for all teams |
| Fix the specific config | Update the template so it's correct by default |
| Escalate the individual delay | Fix the process that causes delays |

### Knowledge Base Updates
When systemic patterns are identified and addressed:
- Update reference architectures and templates
- Add automated checks for known failure patterns
- Create or update training materials
- Add pattern descriptions to onboarding documentation
- Update estimation templates with correction factors
- Add new checklist items to review processes
- Document the pattern, root cause, and fix for future reference

### Pattern Tracking Format
| Pattern | Category | Occurrences | First Seen | Root Cause | Systemic Fix | Status |
|---|---|---|---|---|---|---|
| Description | Process/Tooling/Skills/etc. | Count | Date | Analysis | Proposed solution | Open/In Progress/Resolved |
