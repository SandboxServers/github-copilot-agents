## CAF Anti-Patterns

### Common Mistakes in Cloud Adoption

These anti-patterns appear repeatedly across organizations adopting Azure. Recognizing
them early prevents costly remediation later.

#### 1. Skipping Strategy — Jumping Straight to Technical

**Problem:** Teams start deploying resources without defining business motivations,
outcomes, or a financial model. Cloud adoption becomes a technology exercise.

**Result:** No way to measure success. Budget overruns with no business justification.
Executive support evaporates when value is unclear.

**Fix:** Spend 2–4 weeks on Strategy phase. Define motivations, measurable outcomes,
and a first adoption project before touching Azure.

#### 2. No Landing Zone — Ad-Hoc Subscriptions

**Problem:** Workloads deployed into unstructured subscriptions with no shared networking,
no consistent RBAC, no policy enforcement.

**Result:** Shadow IT, inconsistent security posture, networking nightmares when workloads
need to communicate. Retrofitting landing zones around live workloads is painful.

**Fix:** Deploy at minimum an MVP landing zone (management groups, hub network, baseline
policies) before first production workload.

#### 3. No Governance — Uncontrolled Spend and Security Gaps

**Problem:** Governance treated as "we'll add it later" or skipped entirely. No budgets,
no tagging, no policy enforcement, no RBAC discipline.

**Result:** Surprise cloud bills, resources in unapproved regions, over-permissioned
accounts, compliance violations discovered during audit.

**Fix:** Implement minimum viable governance on day one: cost alerts, required tags (audit
mode), allowed regions, no broad Owner assignments.

#### 4. Lift-and-Shift Everything

**Problem:** Every workload rehosted to VMs without evaluating alternatives. No
rationalization using the 8 Rs.

**Result:** Running the same inefficient architecture in cloud at higher cost. Missing
PaaS benefits. "Cloud is more expensive than on-prem" narrative takes hold.

**Fix:** Assess each workload individually. Rehost where appropriate, but actively
identify replatform, refactor, replace, and retire candidates.

#### 5. Big Bang Migration

**Problem:** Attempting to migrate the entire estate in one massive effort rather than
phased waves.

**Result:** Overwhelmed teams, compounding issues, no time to learn from mistakes. Single
failure can derail the entire program.

**Fix:** Organize into migration waves. Start with low-risk workloads to build confidence
and refine processes. Increase complexity with each wave.

#### 6. No Organizational Readiness

**Problem:** Cloud adoption launched without addressing skills gaps, operating model,
or organizational structure.

**Result:** Teams don't know their roles. Central IT becomes a bottleneck. Developers
bypass governance because it's unclear or too slow.

**Fix:** Define the operating model (centralized, federated, CCoE). Identify skills gaps
and invest in training. Establish RACI before migration begins.

#### 7. Treating CAF as a Checkbox

**Problem:** CAF documents completed as a compliance exercise — lengthy documents produced
but not used to drive decisions.

**Result:** Beautiful architecture documents that don't match reality. Teams ignore
guidance because it wasn't built collaboratively.

**Fix:** Use CAF as a living framework. Reference it for specific decisions. Update
governance and architecture as the environment evolves. CAF is a compass, not a map.
