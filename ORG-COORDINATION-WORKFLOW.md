# Cloud Engineering Org — File-Based Coordination Workflow

**Goal**: Prevent context bloat and create an audit trail that enables gap analysis between specification and final delivery.

**Key Insight**: Each agent call is fresh context. Files are the handoff medium, not conversation history.

---

## Engagement Lifecycle

### Phase 1: Scoping (Org Agent)

**Org reads user request, asks scoping questions**:
- Greenfield or brownfield?
- Timeline?
- Budget?
- Compliance frameworks?
- Team capabilities?
- Success criteria?

**Org creates engagement folder**:
```
engagement-[name]/
├── SCOPE.md                    # Captured answers
├── ARCHITECTURE-PLAN.md        # (created after assessments)
├── AGENT-CALLS.json            # (created after first call)
└── outputs/                    # (created as divisions report)
```

---

### Phase 2: Division Assessment (Fresh Context Per Agent)

**Org calls Platform Engineering Lead** (FRESH CONTEXT):

```markdown
Prompt to @platform-engineering-lead:

---

# PLATFORM ASSESSMENT

Engagement: enterprise-migration  
Phase: Assessment  
Task: Assess landing zone readiness, governance approach  

Read: `engagement-enterprise-migration/SCOPE.md`  
Write to: `engagement-enterprise-migration/outputs/platform-outputs.md`  

Return: [path] + summary of 3 bullets

---

[Brief context from SCOPE.md]
```

**Platform Lead responds**:
```
Created: engagement-enterprise-migration/outputs/platform-outputs.md

Summary:
- Recommended ALZ pattern with 3-tier management groups
- RBAC model: service principal per workstream
- Ready to proceed; no blockers
```

**Org logs this call** to AGENT-CALLS.json:
```json
{
  "sequence": 1,
  "agent": "Platform Engineering Lead",
  "timestamp": "2024-03-15T10:30:00Z",
  "prompt_summary": "Assess landing zone readiness, governance gaps",
  "output_path": "outputs/platform-outputs.md",
  "agent_response": "ALZ pattern, 3-tier mgmt groups, RBAC strategy",
  "what_made_into_plan": ["3-tier mgmt groups", "service principal RBAC"],
  "what_was_rejected": [],
  "status": "incorporated"
}
```

---

**Org calls Security & Compliance Analyst** (FRESH CONTEXT):

```markdown
Prompt to @security-compliance-analyst:

---

# SECURITY ASSESSMENT

Engagement: enterprise-migration  
Phase: Assessment  
Task: Identify compliance requirements, security controls  

Read: `engagement-enterprise-migration/SCOPE.md`  
Write to: `engagement-enterprise-migration/outputs/security-review.md`  

Return: [path] + summary

---

[Context from SCOPE.md]
```

**Security responds**:
```
Created: engagement-enterprise-migration/outputs/security-review.md

Summary:
- HIPAA required (scope indicates healthcare data)
- Recommended: encryption at rest/transit, audit logging, CA baseline
- Threat model: 3 key risks, mitigations outlined
```

**Org logs**:
```json
{
  "sequence": 2,
  "agent": "Security & Compliance Analyst",
  "prompt_summary": "Identify compliance, security controls",
  "output_path": "outputs/security-review.md",
  "agent_response": "HIPAA required. Encryption, audit, CA baseline.",
  "what_made_into_plan": [
    "HIPAA compliance",
    "Encryption requirements",
    "Conditional Access baseline"
  ],
  "what_was_rejected": [],
  "status": "incorporated"
}
```

---

**Org calls other divisions** (Azure Specialty Lead, DevOps Lead, Cost Specialist, etc.)

Each call is:
- **Fresh context** — no prior agent responses in the prompt
- **Specific task** — what to assess, what to write, where
- **File-based input** — read SCOPE.md and any prior outputs
- **File-based output** — write to a specific path
- **One response** — agent responds once, then moves to next task

---

### Phase 3: Create Specification (Org Agent)

**Org synthesizes all division outputs** into `ARCHITECTURE-PLAN.md`:

```markdown
# Architecture Plan — Enterprise Migration

## User Request (from SCOPE.md)
[Summary of what user is trying to accomplish]

## Architecture Approach
[What we're building and why]

## Technology Decisions
- Landing Zones: Azure Landing Zones (ALZ) pattern, 3-tier mgmt groups
  - Why: Compliance isolation, RBAC clarity, scalability (from Platform assessment)
- Identity: Managed identity + RBAC (service principals per workstream)
  - Why: No secrets, zero-trust alignment (from Security assessment)
- Networking: Hub-spoke with private endpoints
  - Why: Compliance isolation, cost optimization (from Specialty assessment)

## Compliance Requirements Met
- HIPAA encryption at rest and in transit ✅
- Audit logging to SIEM ✅
- Conditional Access baseline ✅
- MFA enforced ✅

## Cost Estimate (Approved)
[From Cost Specialist assessment]
- Monthly: $X
- Year 1: $Y
- Assumptions: [listed]

## Security Sign-Off
[From Security assessment]
- Approved: Yes, conditional on audit logging implementation

## Phase Boundaries
- Phase 1 (weeks 1-2): Platform foundation
- Phase 2 (weeks 3-4): Identity & governance  
- Phase 3 (weeks 5-6): Services deployment
- Phase 4 (week 7): DevOps & automation
- Phase 5 (week 8): Testing & go-live

## Division Assignments
| Division | Phase | Component | Owner |
|----------|-------|-----------|-------|
| Platform | 1-2 | Landing zones, mgmt groups, RBAC | Platform Lead |
| Specialty | 3 | Services (App Service, DB, storage) | Azure Specialty Lead |
| Identity | 2 | Entra ID, managed identity, PIM | Identity Lead |
| DevOps | 4 | Pipelines, IaC for deployment | DevOps Lead |
| Security | All | Reviews, gates, hardening | Security Analyst |

## Known Risks & Mitigations
| Risk | Mitigation |
|------|-----------|
| Compliance review delay | Engage compliance team in Phase 1 |
| Skill gap on IaC | Training + pair programming with DevOps |
| Budget overrun | Cost check at each phase gate |
```

**This is the engagement specification** — the contract with the user about what will be delivered.

---

### Phase 4: Implementation (Fresh Context Per Agent)

**Org calls Code Authors** (Terraform, Bicep, PowerShell, etc.)

```markdown
Prompt to @azure-terraform-author:

---

# IMPLEMENTATION TASK

Engagement: enterprise-migration  
Phase: 4-Implementation  
Component: Infrastructure as Code (Terraform)  
Task: Build Terraform for landing zones, networking, identity  

Read:
- `engagement-enterprise-migration/ARCHITECTURE-PLAN.md` (design intent)
- `engagement-enterprise-migration/outputs/platform-outputs.md` (platform requirements)
- `engagement-enterprise-migration/outputs/security-review.md` (security constraints)

Write to: `engagement-enterprise-migration/code/infrastructure/`

Definition of Done:
- [ ] All landing zone resources defined
- [ ] Naming convention: [see naming-conventions skill]
- [ ] No hardcoded secrets (use Key Vault references)
- [ ] Security controls from ARCHITECTURE-PLAN implemented
- [ ] Tests included
- [ ] README with setup instructions

Return: [path] + summary of what was built

---

[Architecture detail sections relevant to Terraform author]
```

**Terraform author responds**:
```
Created:
- engagement-enterprise-migration/code/infrastructure/main.tf
- engagement-enterprise-migration/code/infrastructure/variables.tf
- engagement-enterprise-migration/code/infrastructure/tests/

Summary:
- Built ALZ pattern: 3 management groups (platform, landing zones, decommissioned)
- 15 subscriptions across 3 landing zones
- Networking: hub/spoke with private endpoints
- RBAC: service principals for CI/CD
- Note: Assumed Terraform 1.5+; pinned provider versions as per plan
```

**Org logs**:
```json
{
  "sequence": 12,
  "agent": "Azure Terraform Author",
  "prompt_summary": "Build Terraform for landing zones, networking, identity",
  "output_path": "code/infrastructure/",
  "agent_response": "15 subscriptions, hub/spoke network, RBAC via SPs",
  "what_made_into_plan": [
    "All infrastructure from architecture plan"
  ],
  "blockers": [],
  "status": "delivered"
}
```

---

**Org calls DevOps Lead** (fresh context):

```markdown
Prompt to @devops-lead:

---

# PIPELINE & IaC STRATEGY

Engagement: enterprise-migration  
Phase: 4-Implementation  
Task: Design CI/CD pipelines for infrastructure deployment  

Read:
- `engagement-enterprise-migration/ARCHITECTURE-PLAN.md`
- `engagement-enterprise-migration/code/infrastructure/` (what's being deployed)
- `engagement-enterprise-migration/outputs/security-review.md` (security gates)

Write to: `engagement-enterprise-migration/code/pipelines/`

---

[Relevant sections from architecture]
```

---

### Phase 5: Review & Approval (Fresh Context Per Specialist)

**Org calls Security for final review** (fresh context):

```markdown
Prompt to @security-compliance-analyst:

---

# FINAL SECURITY REVIEW

Engagement: enterprise-migration  
Phase: 5-Review  
Task: Verify final implementation meets security specification  

Read:
- `engagement-enterprise-migration/ARCHITECTURE-PLAN.md` (spec)
- `engagement-enterprise-migration/outputs/security-review.md` (original requirements)
- `engagement-enterprise-migration/code/` (actual implementation)

Write to: `engagement-enterprise-migration/outputs/security-final-review.md`

Question: Does the final implementation meet the specification?
- Encryption at rest/transit: ✅/❌
- Audit logging: ✅/❌
- Conditional Access: ✅/❌
- [List others]

Approval: Yes / Conditional / No

---

[Specification details]
```

**Security responds**:
```
Created: engagement-enterprise-migration/outputs/security-final-review.md

Approval: Yes, conditional on:
- Audit logging configured before go-live
- Threat model review with customer team

Summary:
- All required controls implemented
- One gap: Key Vault soft-delete not configured (low risk, easy to fix)
- Recommended: Enable diagnostic logging before production
```

---

### Phase 6: Final Delivery & Retrospective

**User delivers final implementation** (code, infrastructure, docs).

**Org creates final-delivery/ folder**:
```
engagement-enterprise-migration/
├── final-delivery/
│   ├── code/                          # Actual delivered code
│   ├── infrastructure/                # Actual deployed infrastructure
│   ├── documentation/                 # Actual deliverables
│   └── FINAL-DELIVERY-MANIFEST.md    # What was built vs spec
```

**Org calls Retrospective Agent** (fresh context, comprehensive):

```markdown
Prompt to @retrospective-agent:

---

# ENGAGEMENT RETROSPECTIVE

Engagement: enterprise-migration  
Phase: Retrospective  
Task: Analyze planned vs actual delivery  

Read these files:
1. `engagement-enterprise-migration/SCOPE.md` — what user requested
2. `engagement-enterprise-migration/ARCHITECTURE-PLAN.md` — what we specified
3. `engagement-enterprise-migration/AGENT-CALLS.json` — all decisions made
4. `engagement-enterprise-migration/outputs/` — all specialist assessments
5. `engagement-enterprise-migration/final-delivery/` — what was actually built

Question: Why does the final implementation differ from the specification?

Analyze:
- Architecture: Did final match spec? If not, why?
- Cost: Estimate vs actual?
- Security: Spec requirements vs implementation?
- Timeline: Planned vs actual?
- Scope: What changed and why?

Output to: `engagement-enterprise-migration/RETROSPECTIVE.md`

Include:
- Planned vs Actual comparison table
- Gap analysis (what diverged and why)
- Decision audit trail (when/why decisions changed)
- Systemic patterns (if this is the 3rd engagement with the same gap...)
- Action items for next engagement
- Organizational learning (updates to templates/processes)

---

[All specification and delivery details]
```

**Retrospective Agent produces**:
```
engagement-enterprise-migration/RETROSPECTIVE.md

---

# RETROSPECTIVE: Enterprise Migration

## Planned vs Actual

| Dimension | Planned | Actual | Delta | Reason |
|-----------|---------|--------|-------|--------|
| Timeline | 8 weeks | 10 weeks | +25% | Storage migration took longer than estimated (data volume 1.5x initial estimate) |
| Cost | $45K | $52K | +16% | Additional data egress costs (10TB vs 5TB estimated) |
| Compliance | All HIPAA controls | All HIPAA + additional PCI (out of scope) | Added scope | Customer requested PCI-DSS for future card processing |
| Architecture | 3 tiers | 3 tiers + 1 sandbox | Added tier | Security requested sandbox environment for testing |

## What Went Well
- Platform foundation: 100% aligned with spec
- Security review: Found 2 issues, both fixed before production
- Cost tracking: Caught overage early, optimization applied

## What Diverged & Why
### Data Migration Timeline (+2 weeks)
- Spec assumed 5TB data
- Actual: 7.5TB (discovered during assessment)
- Reason: Customer hadn't done full inventory
- Impact: Delayed go-live, increased labor costs
- Lesson: Require data audit before scoping

### Additional PCI Controls (added scope)
- Not in original scope, added during Phase 3
- Customer business case changed (new feature needs card processing)
- Security correctly escalated
- Result: Architecture updated, Phase 5 extended
- Lesson: Re-scope conversations should trigger re-estimate

### Sandbox Environment (architecture change)
- Not in ARCHITECTURE-PLAN.md
- Security required for testing controls
- Reasonable request; builds confidence
- Result: Minor infrastructure addition
- Lesson: "Security testing environment" should be in default template

## Decision Audit Trail (from AGENT-CALLS.json)
1. Platform assessed: 3-tier, hub-spoke → INCORPORATED
2. Security assessed: HIPAA controls, no PCI → PARTIAL (PCI added later)
3. Cost estimated $45K based on 5TB → DIVERGED (7.5TB actual)
4. Phase 3: Customer requests PCI → SCOPE CHANGE
5. Security reviews Phase 3: Requests sandbox → NEW DELIVERABLE
6. Storage author builds: +1TB capacity for testing → COST IMPACT

## Systemic Patterns
**Pattern 1: Data Volume Underestimation**
- This is the 3rd engagement with data volume divergence
- Average divergence: +40%
- Mitigation: Require storage/data audit in scoping, add 25% buffer to estimates

**Pattern 2: Mid-Engagement Scope Additions**
- 4 of 5 recent engagements saw scope changes
- Average scope change: +15% effort, +20% cost
- Mitigation: Build in "flexibility gates" at Phase 2; treat scope changes as re-scopes with re-estimates

**Pattern 3: Security Additions Are Good**
- Security always catches something real; average 2-3 findings per engagement
- Lesson: This is working correctly, not a failure

## Action Items for Next Engagement
| Action | Owner | Due | Success Criteria |
|--------|-------|-----|------------------|
| Require data audit/inventory in scoping | Platform Lead | Before next engagement (Sprint 47) | Template updated, used in next 3 engagements |
| Add "flexibility gates" for scope changes | Org Lead | Sprint 48 | Process document + updated engagement template |
| Add security sandbox to platform template | Security Analyst | Sprint 48 | Included in next 5 engagements |
| Document "storage underestimation" pattern | Documentation | Sprint 47 | Lessons learned KB article |

## Organizational Learning
**Updated Templates**:
- Scoping template: Add "data audit requirement" section
- Platform template: Add "security testing sandbox" as standard
- Cost estimation: Add 25% buffer for data/storage
- Engagement template: Add "scope flexibility gates" at Phase 2 and 3

**Next Engagement Improvements**:
- Front-load data audits in scoping (avoids Phase 1-2 surprises)
- Plan for scope change (it happens; treat proactively)
- Security sandbox is standard (not optional add-on)

---

## Approval
- [x] Org Lead reviewed
- [x] Action items assigned
- [x] Lessons fed back into templates
```

---

## Key Benefits of This Pattern

### 1. Context Stays Light
- Each agent conversation: ~5 exchanges
- Org context: Just file paths, not history
- No nested conversation chains

### 2. Audit Trail for Gap Analysis
- Every call logged in AGENT-CALLS.json
- Decision rationale visible
- Why something was/wasn't incorporated
- When scope changed and why

### 3. Specification is Contract
- ARCHITECTURE-PLAN.md is what user agreed to
- When delivery diverges, it's intentional (logged)
- Retrospective can see entire decision chain

### 4. Patterns Become Visible
- "We always underestimate data migration time"
- "Security always finds 2-3 issues"
- "Scope creep happens in Phase 3"
- Action items feed back to improve next engagement

### 5. Retrospective Has All Context
- Receives SCOPE.md (original request)
- Receives ARCHITECTURE-PLAN.md (what we promised)
- Receives AGENT-CALLS.json (all decisions)
- Receives final-delivery/ (what was built)
- Can analyze complete gap without context bloat

---

## File Structure Reference

```
engagement-[name]/
├── SCOPE.md                         # User request & answers (created by Org)
├── ARCHITECTURE-PLAN.md             # Spec (created by Org after assessments)
├── AGENT-CALLS.json                 # Audit log of all agent calls
├── IMPLEMENTATION-TASKS.md          # What each team will build
├── AGENT-CALLS.json                 # Every call: prompt, response, what incorporated
│
├── outputs/
│   ├── platform-outputs.md          # Platform engineering assessment
│   ├── specialty-outputs.md         # Azure specialty assessment
│   ├── identity-outputs.md          # Identity/M365 assessment
│   ├── devops-outputs.md            # DevOps assessment
│   ├── security-review.md           # Security assessment & findings
│   ├── cost-review.md               # Cost estimate & approval
│   ├── security-final-review.md     # Final security sign-off
│   └── cost-final-review.md         # Final cost vs actual
│
├── code/
│   ├── infrastructure/              # Terraform, Bicep
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── tests/
│   ├── pipelines/                   # CI/CD YAML
│   ├── automation/                  # PowerShell, scripts
│   └── README.md                    # How to deploy
│
├── final-delivery/                  # (Populated when user provides implementation)
│   ├── code/
│   ├── infrastructure/
│   ├── documentation/
│   └── FINAL-DELIVERY-MANIFEST.md
│
└── RETROSPECTIVE.md                 # (Created by Retrospective Agent)
    ├── Planned vs Actual
    ├── Gap Analysis
    ├── Decision Audit Trail
    ├── Systemic Patterns
    ├── Action Items
    └── Organizational Learning
```

---

## Org Agent's Workflow Summary

1. **Scope** → ask questions, create SCOPE.md
2. **Assess** → call divisions (fresh context), get outputs, log calls
3. **Plan** → synthesize outputs into ARCHITECTURE-PLAN.md
4. **Implement** → call code authors (fresh context), log calls
5. **Review** → call reviewers (fresh context), log approvals
6. **Deliver** → user provides implementation
7. **Retrospect** → call Retrospective Agent with spec + delivery + decision log
8. **Learn** → feed action items into next engagement templates

Each step creates/updates files. No conversation history bloat. Complete audit trail for gap analysis.
