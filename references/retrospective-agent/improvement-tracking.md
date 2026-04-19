## Improvement Tracking

### From WAF Lessons Learned Framework
1. **Work item creation** — every action item becomes a tracked work item with assigned owner, deadline, and progress tracking
2. **Security roadmap integration** — findings feed into organizational security roadmap
3. **Metrics and KPI development** — KPIs based on lessons learned to measure improvement effectiveness
4. **Training and awareness updates** — update training and documentation based on findings
5. **Tabletop exercise enhancement** — integrate scenarios into future exercises

### Action Item Quality
**Bad action items** (vague, unmeasurable):
- "Communicate better"
- "Be more careful with deployments"
- "Improve testing"
- "Review architecture more thoroughly"

**Good action items** (specific, assignable, measurable):
- "Add egress cost estimation to the architecture review template — Owner: Cost Specialist — Due: Sprint 47"
- "Add private endpoint validation to the infrastructure test suite — Owner: Testing Engineer — Due: 2 weeks"
- "Create runbook for DNS resolution failures in hybrid environments — Owner: Network Engineer — Due: Sprint 48"
- "Add deployment stack deny-settings validation to the PR checklist — Owner: Platform Lead — Due: Next release"

### Follow-Up Tracking
Previous retrospective action items must be reviewed at the start of each new retrospective:
- **Completed**: Validate the improvement actually had impact
- **In progress**: Check for blockers, adjust timeline if needed
- **Not started**: Escalate or re-prioritize — if it's perpetually deferred, either do it or explicitly decide not to
- **Abandoned**: Document why — the decision not to act is itself a lesson
