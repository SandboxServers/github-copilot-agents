## DevOps Gotchas — Subtle Traps That Bite You

### Overview
These are not anti-patterns — they are subtle interactions, trade-offs, and
edge cases that catch experienced practitioners off guard. Each one looks
reasonable on the surface but produces unexpected outcomes at scale.

> **Source:** Informed by Microsoft DevOps guidance
> ([DORA metrics](https://learn.microsoft.com/devops/dora/dora-metrics-and-indicators),
> [shift-left testing](https://learn.microsoft.com/devops/develop/shift-left-make-testing-fast-reliable),
> [DevOps security](https://learn.microsoft.com/security/benchmark/azure/mcsb-v2-devop-security)).

### DORA Metrics — Gaming the Numbers vs Improving Outcomes
- Teams optimize for the metric, not the behavior the metric measures
- No-op deployments inflate deployment frequency without delivering value
- Incidents go unreported to keep MTTR artificially low
- Change failure rate drops because teams stop deploying risky changes
- **Gotcha:** Metrics must be used for learning, not judgment. Share transparently,
  celebrate improvement trends, and never tie DORA numbers to performance reviews.

### Pipeline Parallelism — Shared Resources Cause Race Conditions
- Parallel pipeline jobs write to the same storage account, database, or file share
- Tests pass individually but fail when two pipelines run concurrently
- Flaky tests blamed on "timing issues" are actually resource contention
- **Gotcha:** Isolate shared state per pipeline run. Use unique resource names, dedicated
  test databases, or containerized dependencies. Parallelism only works with isolation.

### Secret Management — Pipeline Variables vs Key Vault
- Secrets stored as pipeline variables feel convenient but lack rotation and auditing
- Variable groups shared across pipelines widen the blast radius of a leak
- Key Vault integration adds latency and requires identity configuration
- **Gotcha:** Pipeline variables are acceptable for non-sensitive config. Secrets belong
  in Key Vault with RBAC, rotation policies, and audit logging. The convenience of
  pipeline variables is not worth the security trade-off for credentials.

### Agent Pools — Microsoft-Hosted vs Self-Hosted Trade-offs
- Microsoft-hosted agents are clean every run but have limited customization and cold starts
- Self-hosted agents are fast and customizable but accumulate state and require maintenance
- Scale-set agents look like the best of both worlds until you hit image update cadence issues
- **Gotcha:** Microsoft-hosted for most workloads. Self-hosted only when you need private
  network access, GPU builds, or very large tool installations. Never let self-hosted agents
  become snowflakes — rebuild images on a schedule, not on demand.

### Deployment Slots — Slot-Specific vs Sticky Settings
- App settings marked "slot setting" stay with the slot during a swap
- App settings NOT marked "slot setting" travel with the app code
- Connection strings that should be slot-specific (pointing to staging DB) follow the
  code into production, breaking the swap silently
- **Gotcha:** Audit every app setting and connection string for slot stickiness before
  enabling slot swaps. Test the swap in a non-production environment first. Document
  which settings are sticky and why.

### Pipeline Triggers — Path Filters and Branch Filters Interact Non-Obviously
- Path filters say "only trigger when these files change"
- Branch filters say "only trigger on these branches"
- Combined: a PR that changes files outside the path filter on a monitored branch
  does NOT trigger — which may skip required validation
- Trigger overrides and batch settings add further complexity
- **Gotcha:** Test trigger configurations with real PRs in a sandbox. Document the
  intended trigger behavior and verify it matches reality. Review trigger logs when
  pipelines don't fire as expected.

### Approval Gates — Too Many Approvals Slow Delivery Without Improving Quality
- Every environment has a manual approval: dev → staging → pre-prod → prod
- Approvers rubber-stamp because they approve 20 times a day
- Lead time inflates while actual review quality degrades
- **Gotcha:** Use approvals sparingly — production only, with meaningful reviewers.
  Replace manual gates with automated quality checks (test results, security scans,
  compliance policies). If an approval adds no information, it adds only delay.
