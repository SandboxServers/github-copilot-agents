## Retrospective Best Practices

### Purpose
These practices are proven to produce consistently valuable retrospectives that drive
real improvement. Apply them as defaults and adapt to team context.

### Blameless Postmortems
Focus on what failed, not who failed.
- Use language that centers on systems: "The deployment process lacked a rollback gate"
- Avoid personal attribution even when an individual action was the proximate cause
- The question is always "What conditions allowed this to happen?"
- Document the systemic fix, not the person who will "be more careful next time"
- Blamelessness enables the honesty required to find real root causes
- Teams practicing blameless retros report more near-miss disclosures over time

### Structured Formats
Use established frameworks to guide discussion and ensure coverage.
- **Start/Stop/Continue** — simple and effective for routine sprint retros
- **4Ls (Liked, Learned, Lacked, Longed For)** — broader emotional range, good for engagement retros
- **Timeline** — chronological walkthrough, essential for incident postmortems
- **Sailboat** — visual metaphor (wind=helps, anchor=hinders, rocks=risks), good for engagement kickoffs
- **Lean Coffee** — participant-driven agenda, effective when the team resists structured formats
- Match the format to the context — don't use a lightweight format for a major incident review
- Rotate formats every 3-4 sessions to maintain engagement

### SMART Action Items
Every action item must be specific, measurable, assigned, realistic, and time-bound.
- **Specific** — "Add egress cost line item to architecture review template" not "improve cost tracking"
- **Measurable** — define success criteria: "Template updated and used in next 2 reviews"
- **Assigned** — a named individual owner, not a team or role
- **Realistic** — achievable within the time frame given current workload
- **Time-bound** — explicit due date, not "soon" or "next sprint"
- Limit to 2-3 action items per retro — fewer items with higher completion rate beats many items with none

### Follow Up on Previous Action Items
Open every retrospective by reviewing the status of prior commitments.
- Review each outstanding action item: completed, in progress, blocked, or abandoned
- Celebrate completions — this reinforces that the retro process produces results
- For blocked items, decide: re-commit with a new approach, escalate, or explicitly abandon
- Track completion rate as a retrospective health metric (target: 70%+ per cycle)
- If items are consistently carried forward, the team is taking on too many or they are too large

### Rotate Facilitators
Don't let one person always run the retrospective.
- Rotation distributes ownership and brings different facilitation styles
- New facilitators ask different questions and notice different patterns
- Provide a facilitation guide so rotation doesn't depend on natural skill
- Pair experienced and new facilitators for knowledge transfer
- The scrum master or team lead should facilitate less often, not more

### Time-Box Sessions
Structure the agenda and enforce time limits.
- 60-90 minutes maximum for a full retrospective
- Suggested breakdown: 5 min safety check, 10 min review prior actions, 25 min gather data,
  20 min generate insights, 15 min decide actions, 5 min close
- Use a visible timer — time pressure encourages focus
- If discussion on a topic exceeds its box, park it for deeper follow-up outside the retro
- Shorter sessions (30-45 min) work for routine sprints with no major incidents

### Capture Decisions in a Searchable System
Retrospective outputs must be findable and referenceable.
- Store retro notes, decisions, and action items in a shared, searchable location
- Link action items to work tracking systems (ADO work items, GitHub issues)
- Tag retro outputs by engagement, team, and theme for cross-retro analysis
- Avoid capturing retro data only in ephemeral tools (chat messages, whiteboard photos)
- Structured storage enables the systemic pattern analysis that makes retros compound in value

### Track Improvement Metrics Over Time
Use data to demonstrate and guide improvement.
- **DORA metrics** — deployment frequency, lead time, change failure rate, recovery time
- **Incident frequency and severity** — trending down indicates process improvement
- **Action item completion rate** — health metric for the retro process itself
- **Recurring theme frequency** — same issues appearing retro after retro signals systemic failure
- Review metric trends quarterly, not just per-retro, to see the bigger picture
- Present trends to the team — visible progress motivates continued engagement

### Celebrate What Went Well
Retrospectives are not just for problems.
- Dedicate explicit time to successes — what worked and why
- Identify practices to replicate: "The architecture review caught the egress risk early"
- Recognition of good work builds the psychological safety needed to discuss failures
- Balance the agenda: aim for roughly equal time on positives and improvement areas
- Successful patterns should be documented and shared, not just assumed to continue

### Safety Check at Start
Verify that participants feel safe to speak honestly before proceeding.
- Use a 1-5 scale: "How safe do you feel to share openly in this session?"
- If average safety is below 3, adjust the format (more anonymity, smaller groups)
- Safety scores below 2 indicate a team or management issue that retros alone can't fix
- Track safety scores over time — declining safety is an early warning signal
- Never force people to explain their safety score publicly
- The facilitator should model vulnerability: share their own concerns first
