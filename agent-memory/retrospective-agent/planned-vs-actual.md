## Planned vs Actual Analysis

### Purpose
Comparing planned deliverables, timelines, and estimates against actual outcomes reveals
systematic gaps in planning, estimation, and execution. This analysis is not about blame
for missed targets — it is about calibrating future plans with evidence.

### What to Compare
| Dimension | Planned | Actual | Variance | Root Cause |
|---|---|---|---|---|
| Timeline | Target dates | Actual delivery dates | Days/weeks delta | Why the gap exists |
| Scope | Committed features | Delivered features | Added/removed items | What drove changes |
| Cost | Budget estimate | Actual spend | Over/under budget | Where estimates missed |
| Quality | Defect targets | Actual defect count | Escaped defects | What testing missed |
| Performance | SLA/SLO targets | Measured performance | Gaps against targets | Architecture decisions |
| Team capacity | Planned velocity | Actual velocity | Capacity variance | Interruptions, context switching |

### Tracking Scope Changes
Document every scope change that occurred after the initial plan:
- What was added and why (new requirements, discovered complexity, regulatory change)
- What was removed and why (deprioritized, blocked, no longer needed)
- What was changed and why (requirements clarified, technical constraints discovered)
- Impact on timeline and cost for each scope change
- Whether scope changes went through a formal change control process

### Estimate Accuracy Analysis
For each major work item, compare estimated effort to actual effort:
- Calculate accuracy ratio (actual / estimated) for each item
- Group by work type: infrastructure, application, security, testing, documentation
- Identify systematic biases: do we consistently underestimate certain types of work?
- Track accuracy trends over multiple sprints or engagements
- Note: perfect estimation is not the goal — reducing variance is

### Common Reasons for Deviation

#### Scope Creep
- Requirements discovered during implementation that were not in the original plan
- Stakeholder requests added mid-engagement without adjusting timeline
- "Small" additions that compound into significant work
- Mitigation: formal change control, explicit trade-off conversations

#### Dependency Delays
- Upstream teams or vendors delivering late
- Access provisioning taking longer than expected
- Approval processes with undefined SLAs
- Mitigation: identify dependencies early, build buffer, escalate proactively

#### Technical Debt
- Existing systems harder to work with than assumed
- Workarounds needed for platform limitations
- Undocumented behaviors discovered during integration
- Mitigation: allocate discovery time, spike risky unknowns first

#### External Factors
- Platform outages or service degradation during the engagement
- Policy or compliance changes mid-project
- Team member availability changes (illness, attrition, reassignment)
- Mitigation: risk register, contingency planning, flexible scheduling

### Velocity Trends
Track team velocity across sprints to identify trends:
- Increasing velocity: team is maturing, removing friction, improving processes
- Stable velocity: sustainable pace established, reliable for forecasting
- Decreasing velocity: investigate — technical debt, morale, process overhead, interruptions
- Erratic velocity: unpredictable work intake, unclear priorities, too many context switches

### Using This for Future Planning
- Adjust estimates based on historical accuracy ratios
- Build in explicit buffer for work types with high variance
- Document estimation assumptions and validate them early
- Use range estimates (best case, likely case, worst case) instead of single-point
- Share planned-vs-actual data with stakeholders to set realistic expectations
- Update estimation templates with lessons learned from each engagement
