## Where CAF Is Right, Overkill, and Where Reality Diverges

### Where CAF Is Right
- **Landing zones before workloads** — every enterprise that skipped this regrets it within 6 months
- **Governance from day one** — retrofitting policy is 10x harder than building it in
- **Management group hierarchy** — the structure is well-proven across thousands of enterprises
- **The 8 Rs framework** — forces honest conversation about each workload's future
- **Iterative approach** — start small, expand. Don't try to boil the ocean.

### Where CAF Can Be Overkill
- **Small organizations (< 50 people)** — full ALZ with hub-spoke may be over-engineered. Start with a simpler foundation.
- **Startup / cloud-native only** — management groups and complex subscription topology add overhead when you have 3 subscriptions
- **The full RACI matrix** — CAF's org structure docs assume large enterprises. Smaller orgs need fewer, multi-hatted roles.
- **Every design area fully addressed before first workload** — perfectionism kills momentum. Ship an MVP landing zone.

### Where Real Implementations Diverge
- **Subscription count** — CAF suggests subscription-per-workload. Many orgs start with fewer, broader subscriptions and refine later.
- **Hub-spoke vs VWAN** — CAF presents both but real choice depends on existing network team skills, not just technical requirements.
- **Governance team vs governance tooling** — smaller orgs implement governance through automation (Policy, Terraform) rather than dedicated teams.
- **Central IT vs CCoE** — most orgs are somewhere between Level 3-5, not cleanly at one level. The maturity model is aspirational.
- **Policy enforcement** — CAF recommends "deny" policies. Real implementations often start with "audit" and graduate to "deny" to avoid breaking things.
