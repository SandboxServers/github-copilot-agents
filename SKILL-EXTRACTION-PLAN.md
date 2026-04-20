# Skill Extraction & Knowledge Reorganization — Complete Action Plan

**Scope**: Audit all 30 agent-memory folders, identify actionable content (checklists, style guides, templates, processes), extract into skills, ensure agent-memory contains only reference material.

**Goal**: Create discoverable, actionable skills that Copilot offers contextually while keeping agent-memory as deep reference material.

**Optimization for LLM**: Design skills with clear structure, decision trees, and checklists. Break large topics into multiple focused skills rather than monolithic ones.

---

## Phase 1: Skill Creation (NEW SKILLS)

### High-Priority Skills (Extract from agent-memory, create immediately)

#### 1. **terraform-style-guide** (from azure-terraform-author)
- **Source**: agent-memory/azure-terraform-author/style-guide.md
- **Action**: Extract style-guide.md, create skills/terraform-style-guide.skill/SKILL.md
- **Content**: Naming conventions, formatting, file layout, commenting, module structure
- **Why**: Marked "ALWAYS load first" — actionable playbook
- **Discovery**: Offer when user shares .tf files or discusses Terraform code style

#### 2. **bicep-style-guide** (from bicep-code-author)
- **Source**: agent-memory/bicep-code-author/style-guide.md
- **Action**: Extract style-guide.md, create skills/bicep-style-guide.skill/SKILL.md
- **Content**: Naming conventions, formatting, decorators, file layout
- **Why**: Marked "ALWAYS load first" — actionable playbook
- **Discovery**: Offer when user shares .bicep files

#### 3. **powershell-style-guide** (from powershell-automation-dev)
- **Source**: agent-memory/powershell-automation-dev/style-guide.md
- **Action**: Extract style-guide.md, create skills/powershell-style-guide.skill/SKILL.md
- **Content**: Naming, commenting, error handling, logging, idempotency, security rules
- **Why**: Marked "Always load first" — actionable playbook
- **Discovery**: Offer when user shares .ps1 files or discusses PowerShell scripts

#### 4. **terraform-code-review** (from azure-terraform-author)
- **Source**: agent-memory/azure-terraform-author/code-quality.md
- **Action**: Extract code-quality.md + anti-patterns.md, create skills/terraform-code-review.skill/SKILL.md
- **Content**: PR review checklist, anti-patterns, security scanning, policy-as-code
- **Why**: Code quality is actionable checklist, not reference
- **Discovery**: Offer when reviewing Terraform PRs

#### 5. **testing-strategy-checklist** (from testing-validation-engineer)
- **Source**: agent-memory/testing-validation-engineer/test-strategy.md
- **Action**: Extract test-strategy.md, create skills/testing-strategy.skill/SKILL.md
- **Content**: Test pyramid, test ratios, shift-left, quality gates configuration
- **Why**: High-reuse, actionable framework
- **Discovery**: Offer when designing testing approach

#### 6. **bicep-code-review** (from bicep-code-author)
- **Source**: agent-memory/bicep-code-author/best-practices.md (security defaults section)
- **Action**: Create skills/bicep-code-review.skill/SKILL.md
- **Content**: Security defaults checklist, identity, private endpoints, diagnostics, project structure review
- **Why**: Best-practices has actionable code review section
- **Discovery**: Offer when reviewing Bicep PRs

#### 7. **api-design-standards** (NEW, cross-agent)
- **Sources**: Scattered in azure-integration-architect, azure-apps-infra-architect
- **Action**: Create skills/api-design-standards.skill/SKILL.md
- **Content**: Contract-first design, versioning, error handling, pagination, throttling, OIDC auth
- **Why**: High-reuse, multiple agents reference API patterns
- **Discovery**: Offer when designing APIs

#### 8. **error-handling-patterns** (NEW, cross-agent)
- **Sources**: azure-integration-architect/failure-design.md, powershell-automation-dev/error-handling.md
- **Action**: Create skills/error-handling-patterns.skill/SKILL.md
- **Content**: Retry patterns, circuit breaker, compensation, idempotency, exponential backoff
- **Why**: Cross-cutting, architectural pattern
- **Discovery**: Offer when handling failures in code

#### 9. **naming-conventions** (NEW, cross-agent)
- **Sources**: All style guides (terraform, bicep, powershell)
- **Action**: Create skills/naming-conventions.skill/SKILL.md
- **Content**: Azure resource naming, variable naming, file structure, constants
- **Why**: High-reuse across code review, consistency
- **Discovery**: Offer when starting new module/function/deployment

#### 10. **documentation-standards** (from documentation-writer)
- **Source**: agent-memory/documentation-writer/ (templates + writing-style)
- **Action**: Create skills/documentation-standards.skill/SKILL.md
- **Content**: README template, ADR template, runbook template, writing style guidelines
- **Why**: Templates are actionable playbooks
- **Discovery**: Offer when starting documentation

### Medium-Priority Skills (Create if time permits, refinement of existing content)

#### 11. **infrastructure-testing-strategy** (from testing-validation-engineer)
- **Source**: agent-memory/testing-validation-engineer/infrastructure-testing.md
- **Action**: Create skills/infrastructure-testing.skill/SKILL.md
- **Content**: Terraform testing (.tftest.hcl), Bicep testing, policy testing, smoke tests
- **Why**: High-value, actionable testing approach
- **Discovery**: Offer when testing IaC

#### 12. **cost-optimization-checklist** (from cost-optimization-specialist)
- **Source**: agent-memory/cost-optimization-specialist/ (multiple files)
- **Action**: Create skills/cost-optimization-checklist.skill/SKILL.md
- **Content**: Right-sizing, commitment discounts, waste elimination, cost analysis framework
- **Why**: Actionable checklist, high-value for cost reviews
- **Discovery**: Offer when optimizing Azure costs

#### 13. **disaster-recovery-planning** (from azure-apps-infra-architect)
- **Source**: agent-memory/azure-apps-infra-architect/disaster-recovery-patterns.md
- **Action**: Create skills/disaster-recovery-planning.skill/SKILL.md
- **Content**: Active-active, active-passive, pilot light, backup & restore, RPO/RTO targets
- **Why**: Actionable patterns, high-impact
- **Discovery**: Offer when designing HA/DR strategies

#### 14. **network-security-design** (from azure-network-engineer + security-compliance-analyst)
- **Source**: agent-memory/azure-network-engineer/, agent-memory/security-compliance-analyst/network-security.md
- **Action**: Create skills/network-security-design.skill/SKILL.md
- **Content**: Network topology, NSG design, private connectivity, WAF/DDoS, least-privilege segmentation
- **Why**: Security review template
- **Discovery**: Offer when designing network security

#### 15. **secrets-management-audit** (from security-compliance-analyst)
- **Source**: agent-memory/security-compliance-analyst/secrets-management.md
- **Action**: Create skills/secrets-management-audit.skill/SKILL.md
- **Content**: Key Vault design, rotation, audit trails, credential handling
- **Why**: Security checklist
- **Discovery**: Offer when reviewing secrets handling

### Low-Priority Skills (Enhancements to existing skills, only if room remains)

#### 16. **database-migration-checklist** (from azure-database-specialist)
- **Source**: agent-memory/azure-database-specialist/migration.md
- **Action**: Create skills/database-migration.skill/SKILL.md

#### 17. **kubernetes-security-hardening** (from azure-compute-engineer + security-compliance-analyst)
- **Source**: AKS deep dive + security frameworks
- **Action**: Create skills/kubernetes-security.skill/SKILL.md

---

## Phase 2: Memory Organization (REORGANIZE existing agent-memory)

### For Each Agent with Extracted Skills:

After extracting content to skills, update agent-memory _toc.md:

1. **azure-terraform-author**: 
   - REMOVE: style-guide.md (moved to terraform-style-guide skill)
   - REMOVE: code-quality.md (moved to terraform-code-review skill)
   - UPDATE: _toc.md to remove references
   - KEEP: anti-patterns, best-practices, provider-landscape, state-management, testing, etc. (reference)

2. **bicep-code-author**:
   - REMOVE: style-guide.md (moved to bicep-style-guide skill)
   - REMOVE: best-practices.md security review section (moved to bicep-code-review skill)
   - UPDATE: _toc.md
   - KEEP: language-fundamentals, modules-avm, what-if-linter, gotchas, etc.

3. **powershell-automation-dev**:
   - REMOVE: style-guide.md (moved to powershell-style-guide skill)
   - UPDATE: _toc.md
   - KEEP: error-handling, module-ecosystem, authentication-patterns, pester-testing, etc.

4. **testing-validation-engineer**:
   - REMOVE: test-strategy.md (moved to testing-strategy skill)
   - REMOVE: infrastructure-testing.md (moved to infrastructure-testing skill)
   - UPDATE: _toc.md
   - KEEP: load-testing, chaos-engineering, test-data-management, anti-patterns, gotchas

5. **documentation-writer**:
   - REMOVE: readme-template, adr-template, runbook-template (moved to documentation-standards skill)
   - UPDATE: _toc.md
   - KEEP: writing-style, audience-adaptation, documentation-types, platforms, gotchas

6. All other agents:
   - **No changes** (their content is reference/deep-dive, stays in agent-memory)

---

## Phase 3: Implementation Sequence (for You to Give to GitHub Copilot)

### Step 1: Create new branch
```bash
git checkout -b feat/skill-extraction
```

### Step 2: Create 10 High-Priority Skills (3-4 hours estimated)

For each of these skills, create `skills/{skill-name}.skill/SKILL.md`:

1. **terraform-style-guide** — Extract from agent-memory/azure-terraform-author/style-guide.md
2. **bicep-style-guide** — Extract from agent-memory/bicep-code-author/style-guide.md
3. **powershell-style-guide** — Extract from agent-memory/powershell-automation-dev/style-guide.md
4. **terraform-code-review** — Combine agent-memory/azure-terraform-author/code-quality.md + anti-patterns.md
5. **testing-strategy** — Extract from agent-memory/testing-validation-engineer/test-strategy.md
6. **bicep-code-review** — Extract security review section from agent-memory/bicep-code-author/best-practices.md
7. **api-design-standards** — New skill synthesizing patterns from azure-integration-architect and azure-apps-infra-architect
8. **error-handling-patterns** — New skill from azure-integration-architect/failure-design.md + powershell-automation-dev/error-handling.md
9. **naming-conventions** — New skill synthesizing all style-guide.md files
10. **documentation-standards** — Combine agent-memory/documentation-writer/ templates

For each skill:
- Create directory: `mkdir -p skills/{skill-name}.skill`
- Create SKILL.md with proper frontmatter (lowercase name, hyphenated)
- Adapt extracted content for LLM readability (checklists, decision trees, bullet points)
- Commit: `feat: add {skill-name} skill`

### Step 3: Remove extracted files from agent-memory (1 hour)

Delete these files:
- `agent-memory/azure-terraform-author/style-guide.md`
- `agent-memory/azure-terraform-author/code-quality.md`
- `agent-memory/bicep-code-author/style-guide.md`
- `agent-memory/powershell-automation-dev/style-guide.md`
- `agent-memory/testing-validation-engineer/test-strategy.md`
- `agent-memory/testing-validation-engineer/infrastructure-testing.md`
- `agent-memory/documentation-writer/readme-template.md`
- `agent-memory/documentation-writer/adr-template.md`
- `agent-memory/documentation-writer/runbook-template.md`

Update these _toc.md files to remove the deleted entries:
- `agent-memory/azure-terraform-author/_toc.md`
- `agent-memory/bicep-code-author/_toc.md`
- `agent-memory/powershell-automation-dev/_toc.md`
- `agent-memory/testing-validation-engineer/_toc.md`
- `agent-memory/documentation-writer/_toc.md`

Commit: `refactor: remove extracted content from agent-memory`

### Step 4: Create Medium-Priority Skills (optional, 2 hours)

If proceeding:
1. **infrastructure-testing** — agent-memory/testing-validation-engineer/infrastructure-testing.md
2. **cost-optimization-checklist** — agent-memory/cost-optimization-specialist/ (synthesis)
3. **disaster-recovery-planning** — agent-memory/azure-apps-infra-architect/disaster-recovery-patterns.md
4. **network-security-design** — agent-memory/azure-network-engineer/ + agent-memory/security-compliance-analyst/network-security.md
5. **secrets-management-audit** — agent-memory/security-compliance-analyst/secrets-management.md

Commit after each 2-3: `feat: add {skill-names}`

### Step 5: Update Documentation (30 mins)

Update `BEST-PRACTICES.md`:
- Add section: "Skill Catalog & When to Use Them"
- List all 10 (or 15) skills with brief descriptions
- Link to each skill

Commit: `docs: document new skills`

### Step 6: Push & Open PR

```bash
git push origin feat/skill-extraction
```

Create PR with this body:
```markdown
## Skills Extraction & Agent-Memory Reorganization

### Summary
- Extracted 10 actionable skills from agent-memory
- Converted style guides, checklists, templates to discoverable skills
- Reorganized agent-memory to contain only reference/deep-dive material

### Skills Created
1. terraform-style-guide
2. bicep-style-guide
3. powershell-style-guide
4. terraform-code-review
5. testing-strategy
6. bicep-code-review
7. api-design-standards
8. error-handling-patterns
9. naming-conventions
10. documentation-standards

### Changes to Agent-Memory
- Removed 14 files (converted to skills)
- Updated 5 agent _toc.md indexes
- Agent-memory now contains only reference material

### Verification
- All skill names are lowercase with hyphens
- All skill descriptions match discovery intent
- No broken links in _toc.md files
- All removed files verified to have no other references in agents/ or CLAUDE.md
```

---

## Skill Content Structure Template

Each skill should follow this template for maximum LLM usability:

```markdown
---
name: skill-name
description: One-line description for Copilot auto-discovery
when-to-use: >
  Bullet list of scenarios when Copilot should offer this skill
categories:
  - category1
  - category2
---

# [Skill Title]

## Quick Reference / TL;DR
- Key decision tree or bullet points
- Most important items first

## Decision Tree (if applicable)
```
Question?
├─ Yes → Action A
└─ No → Question 2?
    ├─ Yes → Action B
    └─ No → Action C
```

## Detailed Checklists

### Section A Checklist
- [ ] Item 1
- [ ] Item 2
- [ ] Item 3

### Section B Checklist
- [ ] Item 1
- [ ] Item 2

## Common Patterns / Examples

### Pattern 1
Code or template example

### Pattern 2
Code or template example

## Anti-Patterns / What NOT to Do

### Mistake 1
Why this is wrong and fix

## Links & Related Skills
- Related skill: [skill-name]
- External docs: [link]
```

---

## Success Criteria

✅ All 10+ high-priority skills created  
✅ Skill names are lowercase with hyphens  
✅ All skill descriptions are clear (Copilot uses for matching)  
✅ Extracted files removed from agent-memory  
✅ All _toc.md files updated  
✅ No broken links (verify removed files have no references elsewhere)  
✅ PR opens with clear description  

---

## Important Notes

1. **Commit frequently**: After each 1-2 skills, commit to remote
2. **Verify link integrity**: Before deleting from agent-memory, grep for references:
   - Check CLAUDE.md
   - Check agents/*.agent.md
   - Check other _toc.md files
3. **Skill names**: Must be lowercase, hyphens only (no spaces, no capitals)
4. **Optimize for LLM**: Use checklists, decision trees, short bullets. Avoid long prose.
5. **Preserve reference material**: Anti-patterns, gotchas, best-practices stay in agent-memory
6. **Push regularly**: Push to remote every 30 mins (after 3-4 commits)

---

## Timeline

- **High-priority skills**: 3-4 hours
- **Memory cleanup**: 1 hour
- **Medium-priority skills** (optional): 2 hours
- **Docs + PR**: 30 mins
- **Total**: 6.5 hours (high-priority only) / 8.5 hours (with medium-priority)

---

## When to Stop / Adjust

If getting complexity/scope creep:
- **Stop after high-priority 10 skills** — they deliver the most value
- **Skip medium-priority** — nice-to-have, can do later
- **Rollback**: `git reset --hard origin/main` if needed

Safe to proceed at whatever pace makes sense.
