---
name: engagement-coordination-protocol
description: >-
  Standardized protocol for multi-agent engagements. Defines engagement folder
  structure, file formats, handoff templates, and how agents coordinate with
  fresh context to prevent information bloat.
when-to-use: >-
  Use when: orchestrating multi-agent engagements, setting up engagement folders,
  calling other agents from an orchestrator role, responding to org/lead agent
  calls, structuring handoffs for fresh context, maintaining audit trails of
  decisions, setting up retrospective analysis.
categories:
  - coordination
  - multi-agent
  - engagement-management
---

# Engagement Coordination Protocol

Standard protocol for multi-agent engagements. Ensures all agents follow the same folder structure, file formats, and handoff expectations—even across different organizations.

---

## Quick Reference

**Use this skill when**:
- You're orchestrating multiple agents (org/lead role)
- You're being called by an orchestrator agent
- You're setting up an engagement
- You're responding to a coordinated work request
- You need to structure handoffs with fresh context

---

## Engagement Folder Structure

Create this folder structure at engagement start. All agents reference these paths.

```
engagement-[name]/
│
├── SCOPE.md
│   What: User request, constraints, success criteria, budget, timeline
│   Owner: Orchestrator agent (initially)
│   Access: All agents read this for context
│
├── ARCHITECTURE-PLAN.md
│   What: Final specification (what we're building, why, constraints met)
│   Owner: Orchestrator agent (synthesized from all division assessments)
│   Created: After all initial assessments complete
│   Purpose: This is the contract with the user. Everything measured against this.
│
├── AGENT-CALLS.json
│   What: Audit log of every agent call
│   Format: [See AGENT-CALLS.json Format section below]
│   Owner: Orchestrator agent (maintains throughout engagement)
│   Purpose: Decision trail for retrospective analysis
│
├── IMPLEMENTATION-TASKS.md
│   What: What each team will build (from ARCHITECTURE-PLAN)
│   Owner: Orchestrator agent
│   Format: | Component | Owner | Deliverable | Due |
│
├── outputs/
│   What: Assessment/review outputs from all agents
│   Structure:
│   ├── [division]-outputs.md          (assessments from division leads)
│   ├── [specialist]-outputs.md        (specialist domain outputs)
│   ├── security-review.md             (security findings)
│   ├── cost-review.md                 (cost estimate + approval)
│   ├── security-final-review.md       (final security sign-off)
│   └── cost-final-review.md           (actual vs estimated cost)
│
├── code/
│   What: Implementation artifacts
│   Structure:
│   ├── infrastructure/                (Terraform, Bicep)
│   ├── pipelines/                     (YAML workflows)
│   ├── automation/                    (PowerShell, scripts)
│   └── README.md                      (how to deploy)
│
├── final-delivery/
│   What: User-provided implementation (created when user delivers)
│   Structure:
│   ├── code/                          (actual delivered code)
│   ├── infrastructure/                (actual deployed infrastructure)
│   ├── documentation/                 (actual deliverables)
│   └── FINAL-DELIVERY-MANIFEST.md    (what was built vs spec)
│
└── RETROSPECTIVE.md
    What: Post-engagement analysis (created by retrospective agent)
    Contents:
    ├── Planned vs Actual comparison
    ├── Gap analysis (why divergences)
    ├── Decision audit trail
    ├── Systemic patterns
    ├── Action items for next engagement
    └── Organizational learning (template updates)
```

---

## AGENT-CALLS.json Format

Orchestrator maintains this log throughout engagement. Every agent call is recorded.

```json
{
  "engagement": "enterprise-migration",
  "calls": [
    {
      "sequence": 1,
      "timestamp": "2024-03-15T10:30:00Z",
      "agent": "Platform Engineering Lead",
      "phase": "assessment",
      "prompt_summary": "Assess landing zone readiness, governance gaps",
      "input_files": ["SCOPE.md"],
      "output_path": "outputs/platform-outputs.md",
      "agent_response_summary": "ALZ pattern, 3-tier mgmt groups, RBAC strategy",
      "what_made_into_plan": [
        "3-tier management group hierarchy",
        "service principal RBAC model"
      ],
      "what_was_rejected": [
        "Multi-subscription vending (deferred to Phase 2)"
      ],
      "blockers": [],
      "status": "incorporated"
    },
    {
      "sequence": 2,
      "timestamp": "2024-03-15T11:00:00Z",
      "agent": "Security & Compliance Analyst",
      "phase": "assessment",
      "prompt_summary": "Identify compliance requirements, security controls",
      "input_files": ["SCOPE.md"],
      "output_path": "outputs/security-review.md",
      "agent_response_summary": "HIPAA required. Encryption, audit, CA baseline.",
      "what_made_into_plan": [
        "HIPAA compliance requirements",
        "encryption at rest/transit",
        "Conditional Access baseline"
      ],
      "what_was_rejected": [],
      "blockers": [],
      "status": "incorporated"
    }
  ]
}
```

**Log fields**:
- `sequence` — Call number (1, 2, 3, etc.)
- `timestamp` — When call was made
- `agent` — Which agent was called
- `phase` — assessment | implementation | review | retrospective
- `prompt_summary` — What was asked (1 line)
- `input_files` — What files agent read
- `output_path` — Where agent wrote output
- `agent_response_summary` — What agent returned (2-3 lines)
- `what_made_into_plan` — Which parts were incorporated into ARCHITECTURE-PLAN.md
- `what_was_rejected` — Which parts were not used and why
- `blockers` — Any blockers agent reported
- `status` — incorporated | rejected | pending | blocked

---

## Handoff Template: Division/Lead Assessment

**Use this when**: Orchestrator calls a division lead for initial assessment

```markdown
# ENGAGEMENT ASSESSMENT REQUEST

**Engagement**: [name]
**Phase**: Assessment
**Requested from**: @[Lead-Name]
**Due**: [Date]

---

## Your Task

Assess this engagement from your division's perspective.

**Read**: `engagement-[name]/SCOPE.md`

**Write to**: `engagement-[name]/outputs/[division]-outputs.md`

---

## What We Need

1. **Readiness assessment** — What's in place, what's missing, what's risky
2. **Recommended approach** — Your solution to this problem
3. **Constraints & assumptions** — What you're assuming, what could break
4. **Dependencies** — What needs to happen first for your division
5. **Blockers** — Anything preventing your division from moving forward

---

## Expected Output Format

Write to: `engagement-[name]/outputs/[division]-outputs.md`

Include sections:
- Executive summary (2 lines max)
- Assessment findings (specific, not generic)
- Recommended approach and rationale
- Known constraints
- Dependencies on other divisions (if any)
- Next recommended step

---

## Return to Orchestrator

Path: `engagement-[name]/outputs/[division]-outputs.md`  
Summary: 2-3 bullets of key findings  
Blockers: Any blockers identified  

---

## Context

[User request, success criteria, why this matters]
```

---

## Handoff Template: Specialist Implementation Task

**Use this when**: Orchestrator calls a specialist to build/implement something

```markdown
# IMPLEMENTATION TASK

**Engagement**: [name]
**Phase**: Implementation - [Component]
**Requested from**: @[Specialist-Name]
**Task**: [Specific deliverable]
**Due**: [Date]

---

## Your Task

Implement [specific component] according to ARCHITECTURE-PLAN.md.

---

## What We Need

**Deliverable**: [Specific code/config/script]
- Location: `engagement-[name]/code/[path]/`
- Format: [Terraform / Bicep / PowerShell / Workflow YAML / etc]
- Includes: [tests / validation / documentation]

---

## Read These Files

1. `engagement-[name]/ARCHITECTURE-PLAN.md` — overall spec (skim for your section)
2. `engagement-[name]/outputs/[relevant-output].md` — your specific requirements
3. `engagement-[name]/AGENT-CALLS.json` — decisions made, why they were made

---

## Implementation Constraints

- Security requirements: [from security-review.md]
- Cost constraints: [from cost-review.md]
- Compliance: [from SCOPE.md]
- Dependencies: [what must exist first]

---

## Definition of Done

Your work is done when:
- [ ] All required resources/code defined
- [ ] Tests or validation included
- [ ] Documentation explains setup and operations
- [ ] Naming conventions follow [style-guide skill]
- [ ] No hardcoded secrets or credentials
- [ ] All requirements from ARCHITECTURE-PLAN met

---

## Return to Orchestrator

Path: `engagement-[name]/code/[path]/`  
Summary: What was built, any assumptions made  
Blockers: Anything requiring different approach  
```

---

## Handoff Template: Review & Approval

**Use this when**: Orchestrator calls reviewer (security, cost, etc.)

```markdown
# REVIEW & APPROVAL REQUEST

**Engagement**: [name]
**Phase**: Review - [Component]
**Requested from**: @[Reviewer-Name]
**Scope**: [What to evaluate]
**Due**: [Date]

---

## Your Task

Review deliverables against the specification. Provide findings and approval decision.

---

## What We're Reviewing

[Code / Architecture / Compliance posture / Cost estimate]

---

## Read These Files

1. `engagement-[name]/ARCHITECTURE-PLAN.md` — the specification
2. `engagement-[name]/outputs/[component].md` — what was implemented
3. `engagement-[name]/code/[path]/` or equivalent — actual deliverable
4. `engagement-[name]/AGENT-CALLS.json` — context on decisions

---

## Review Criteria

1. **Does it match specification?** If not, what's different and why?
2. **Requirements met?** Are SCOPE.md requirements satisfied?
3. **Gaps or risks?** Any missing pieces or concerning patterns?
4. **Approval?** Yes / Conditional / No

---

## Return to Orchestrator

Path: `engagement-[name]/outputs/[review-type]-review.md`

Include sections:
- Compliance with specification (yes/no/partial + details)
- Findings (what works, what doesn't, what's risky)
- Recommendation (approve / conditional / reject)
- Conditions/blockers (if applicable)

Return: [path] + summary (approve/conditional/reject + reason)
```

---

## How Agents Respond to Handoffs

When you receive a handoff from an orchestrator:

1. **Read the input files specified in the prompt** (usually SCOPE.md and prior outputs)
2. **Write your output to the exact path specified** (e.g., `engagement-name/outputs/platform-outputs.md`)
3. **Use the section format specified** in the handoff
4. **Return the path + summary** (2-3 bullets)
5. **Report any blockers** that prevent your division from proceeding
6. **Don't ask for clarification on other divisions' work** — that's the orchestrator's job

---

## For Orchestrators: Key Practices

### Before Calling an Agent

- ✅ Create engagement folder and SCOPE.md
- ✅ Have the output path ready
- ✅ Know what files the agent should read
- ✅ Be specific about expected sections/format

### When Agent Responds

- ✅ Log the call to AGENT-CALLS.json
- ✅ Parse the response and note what made it into the plan
- ✅ Note what was rejected and why
- ✅ Store the output file path
- ✅ Call the next agent with fresh context (not prior agent's output)

### At Engagement End

- ✅ Finalize ARCHITECTURE-PLAN.md (this is the spec/contract)
- ✅ Complete AGENT-CALLS.json (full decision trail)
- ✅ When user delivers implementation, call Retrospective Agent with:
  - SCOPE.md (what was requested)
  - ARCHITECTURE-PLAN.md (what was promised)
  - AGENT-CALLS.json (why decisions were made)
  - final-delivery/ (what was actually built)

---

## Key Principles

1. **Fresh context per call** — Each agent gets a focused prompt, not history
2. **Files are the handoff medium** — Not conversation history
3. **One response per agent** — They respond once, move to next
4. **Specification is the contract** — ARCHITECTURE-PLAN.md is measured against
5. **Audit trail for learning** — AGENT-CALLS.json enables retrospective analysis
6. **Engagement structure is consistent** — Same across all engagements
