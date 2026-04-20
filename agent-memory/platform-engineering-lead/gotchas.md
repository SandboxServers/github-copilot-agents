## Platform Engineering Gotchas

### Overview
These are the non-obvious traps that catch even experienced platform teams. Unlike
anti-patterns (which are clearly wrong), gotchas are situations where the right approach
has hidden complexity that surprises teams mid-execution.

### Developer Experience Surveys: What Devs Say vs What They Do
- Developers report wanting flexibility, then ignore options and use defaults
- Survey responses skew toward what sounds good, not actual behavior
- Teams say they want self-service but continue filing tickets out of habit
- **Mitigation:** Combine surveys with telemetry. Measure what developers actually
  do — page views, template usage, time-to-deploy — alongside what they say.
  Behavior data always outranks opinion data.

### Golden Paths: Too Rigid or Too Flexible
- Too rigid: developers hit edge cases, build workarounds, and bypass the platform
- Too flexible: no standardization, every team assembles a bespoke workflow
- The sweet spot shifts as the organization matures
- **Mitigation:** Design golden paths with clear defaults and documented escape hatches.
  Track deviation rates — high deviation means the path needs updating, not that
  developers are wrong. Revisit rigidity quarterly.

### Platform Team Staffing: Infrastructure Alone Isn't Enough
- Teams staffed entirely with infrastructure engineers build great infra, poor UX
- Developer experience requires product thinking, UX skills, and empathy
- Missing developer advocates means the platform optimizes for the wrong audience
- **Mitigation:** Staff the platform team with a mix of infrastructure engineers,
  developer experience specialists, and at least one product manager. Rotate
  developers from stream-aligned teams through the platform team for empathy.

### Self-Service Portals: Maintenance Burden Is Higher Than Expected
- Building the portal is 20% of the effort; maintaining it is 80%
- Every backend system change requires portal updates
- Stale portal data erodes developer trust faster than no portal at all
- **Mitigation:** Use an extensible provider model (Backstage plugins, custom providers)
  so backend teams maintain their own integrations. Automate freshness checks.
  Budget ongoing maintenance at 2-3x the initial build effort.

### Template Evolution: Updating Templates Doesn't Update Existing Instances
- New template version ships, but existing repos stay on the old version
- Drift accumulates — security patches, best practices, dependency updates
- Teams on old templates eventually hit issues the platform team already solved
- **Mitigation:** Design templates with update mechanisms from the start. Use
  scaffolding tools that support upgrades (Yeoman, Cookiecutter with diffing).
  Publish migration guides for breaking changes. Track template version adoption
  and nudge teams on outdated versions.

### Multi-Cloud Support: Doubles Effort for Questionable Value
- Supporting two clouds doubles golden path engineering and testing effort
- Abstraction layers that hide cloud differences also hide cloud strengths
- Most organizations use one cloud for 90%+ of workloads
- **Mitigation:** Build golden paths for the primary cloud first. Only invest in
  multi-cloud when there is a genuine business requirement (regulatory, M&A).
  Abstract at the interface level, not the implementation level.

### Platform as Identity: Team Defends the Platform Instead of Improving It
- Platform team becomes emotionally invested in their architecture
- Criticism of the platform is taken personally instead of as product feedback
- Technical decisions are defended on sunk-cost grounds
- **Mitigation:** Establish a culture of learning over defending. Run regular
  retrospectives focused on developer pain points, not platform features.
  Celebrate pivots and deprecations alongside launches.

### Governance as Afterthought: Bolted On Instead of Built In
- Platform launches without policy-as-code, then governance is added later
- Retrofitting governance breaks existing workflows and frustrates teams
- Developers who were early adopters feel punished for adopting early
- **Mitigation:** Bake governance into golden paths from the start. Use guardrails
  (Azure Policy, OPA, Sentinel) that guide rather than block. Grandfather
  existing deployments with a migration timeline.

### Summary
Gotchas are the gap between planning and reality. The common thread is that
platform engineering requires ongoing investment in understanding how developers
actually use the platform — not just how the platform team designed it to be used.
