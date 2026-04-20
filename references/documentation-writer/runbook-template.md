# Runbook Template

A runbook is a step-by-step procedure for handling a specific operational scenario. It exists so that the person at 2am — tired, stressed, and unfamiliar with this particular system — can resolve the issue without guessing. Every runbook should be copy-pasteable, verifiable, and reversible.

## Runbook Template

```markdown
# Runbook: [Service/Component] — [Scenario]

## Metadata
- **Last Verified**: YYYY-MM-DD
- **Owner**: [Team or person responsible for keeping this current]
- **Review Cadence**: Quarterly

## Trigger
What alert, symptom, or situation triggers this runbook.
Include the exact alert name or error message if applicable.

## Prerequisites
- [ ] Access to [system/tool]
- [ ] Permissions: [specific role or RBAC assignment]
- [ ] On-call contact: [team/person and how to reach them]
- [ ] Tools installed: [CLI, kubectl, etc.]

## Steps

### 1. Diagnose
1. Open [dashboard name] at [exact URL]
2. Run:
   \`\`\`bash
   <exact diagnostic command>
   \`\`\`
   Expected output: [what healthy looks like]
3. If [condition A], proceed to Step 2. If [condition B], escalate to [team].

### 2. Resolve
1. Run:
   \`\`\`bash
   <exact resolution command>
   \`\`\`
   Expected output: [what success looks like]
2. Wait [duration] for [condition] to stabilize.
3. Proceed to Verification.

## Verification
How to confirm the issue is resolved:
1. Check [metric/dashboard] — value should be [expected state]
2. Run:
   \`\`\`bash
   <verification command>
   \`\`\`
   Expected output: [what confirms resolution]
3. Monitor for [duration] to confirm stability.

## Rollback
If resolution fails or makes things worse:
1. Run:
   \`\`\`bash
   <exact rollback command>
   \`\`\`
2. Verify rollback: [how to confirm original state is restored]
3. Escalate to [team/person] with context of what was attempted.

## Escalation
- **Level 1**: [Team] — [contact method] — [when to escalate]
- **Level 2**: [Team/person] — [contact method] — [when to escalate]
- **Emergency**: [person/process for critical escalation]

## Post-Incident
- [ ] Update incident channel with resolution summary
- [ ] Schedule post-mortem within 48 hours
- [ ] Update this runbook if any steps were wrong or incomplete
```

## Runbook Rules

- **Copy-pasteable steps** — every command must be exact. "Run the script" is bad. "Run `./scripts/restart-service.ps1 -Environment prod`" is good
- **Include expected output** — after every command, tell the reader what success looks like. Unexpected output means something is wrong
- **Include rollback for every change** — every action that modifies state needs a way back. No exceptions
- **No tribal knowledge** — if it requires knowledge not in the runbook, the runbook is incomplete
- **Test quarterly** — have someone who didn't write it follow the steps. Fix every place they get stuck
- **Mark placeholders clearly** — use `<PLACEHOLDER>` format, never realistic-looking fake values
- **Date everything** — include "Last Verified" date. A runbook verified 6 months ago may be dangerously wrong today
- **One scenario per runbook** — don't combine multiple procedures. Link between them if needed

## Common Mistakes

- Commands that aren't copy-pasteable (wrong paths, missing arguments, unresolved variables)
- Missing verification steps — the reader completes the runbook but doesn't know if it worked
- No rollback procedure — the reader made things worse and has no way back
- Assumptions about environment access — the reader doesn't have the right permissions at 2am
- Stale runbooks — infrastructure changed but the runbook wasn't updated
- Runbooks that reference other runbooks in a chain without clear entry/exit points
