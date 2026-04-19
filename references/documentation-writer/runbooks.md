## Runbook Best Practices

### Incident Runbook Template

```markdown
# Runbook: [Service/Component] — [Scenario]

## When to Use
[What alert, symptom, or situation triggers this runbook]

## Impact
[What's broken, who's affected, SLA implications]

## Prerequisites
- [ ] Access to [system/tool]
- [ ] Permissions: [specific role/access]
- [ ] On-call contact: [team/person]

## Diagnosis Steps
1. Check [dashboard/metric] at [URL]
2. Run: `[exact command]`
3. If [condition], proceed to Step 4. If not, escalate to [team].

## Resolution Steps
1. [Exact step with exact command]
2. [Next step]
3. Verify resolution: [how to confirm it's fixed]

## Rollback
If resolution fails or makes things worse:
1. [Rollback step]
2. [Rollback step]
3. Escalate to [team/person]

## Post-Incident
- [ ] Update incident channel
- [ ] Schedule post-mortem within 48 hours
- [ ] Update this runbook if steps were wrong or incomplete
```

### Runbook Rules
1. **Write for 2am** — assume the reader is tired, stressed, and unfamiliar with this specific system
2. **No ambiguity** — "Run the script" is bad. "Run `./scripts/restart-service.ps1 -Environment prod`" is good
3. **Include exact commands** — copy-pasteable, with placeholders clearly marked as `<PLACEHOLDER>`
4. **Include verification steps** — after each action, how does the reader know it worked?
5. **Include rollback** — every action that changes state needs a way back
6. **Test the runbook** — have someone who didn't write it follow it. Fix every place they get stuck.
7. **Keep current** — a runbook that was accurate six months ago may kill your SLA today
