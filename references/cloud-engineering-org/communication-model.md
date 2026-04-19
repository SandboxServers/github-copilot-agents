## Communication Model

### Audience-Specific Communication

| Audience | What They Need | How to Communicate |
|----------|---------------|-------------------|
| **Executives** | Outcomes, timelines, risks, budget | "Phase 2 completes in 3 weeks. We're on budget. One risk: compliance review may extend Phase 3 by a week." |
| **Engineers** | Technical specifics, architecture decisions, implementation details | "We're using AKS with KEDA for the event-driven workloads. Network engineer has the VNet design ready for review." |
| **Project Managers** | Workstreams, dependencies, milestones, blockers | "Three parallel workstreams in Phase 3. Database track is blocked on network topology — expected unblock Thursday. Critical path runs through the integration workstream." |

### Escalation Protocol

1. **Division-level blocker**: Lead resolves within their division
2. **Cross-division blocker**: Org Lead mediates between leads
3. **Mandatory gate blocker**: Security/Cost/Docs blocker — the gate authority decides, Org Lead adjusts the plan
4. **External blocker**: Customer decision needed, Azure limitation discovered, timeline constraint — Org Lead communicates to stakeholders and adjusts
