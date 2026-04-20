## Architecture Review

Architecture reviews ensure solutions meet quality standards before they reach production. The Specialty Lead facilitates reviews, ensuring all domains are represented and no pillar is ignored because a specialist is focused on their own area.

### Well-Architected Framework Checklist

Every review evaluates all five pillars:

**Reliability:**
- Availability zones used for critical resources?
- Failover tested — not just configured?
- Backup and restore validated with actual recovery?
- Retry policies and circuit breakers in application code?
- Health probes configured and tested on load balancers?

**Security:**
- Managed identity used everywhere possible?
- No secrets in code, environment variables, or app settings (Key Vault references instead)?
- Private endpoints on all PaaS resources?
- WAF on all public-facing endpoints?
- Defender for Cloud recommendations addressed?
- RBAC follows least privilege — no standing Owner/Contributor on production?

**Cost Optimization:**
- Right-sized SKUs based on actual load (not guesses)?
- Reserved instances or savings plans for predictable workloads?
- Auto-scaling configured with appropriate min/max?
- Storage lifecycle policies for aging data?
- Egress costs understood and minimized (co-locate where possible)?

**Operational Excellence:**
- All infrastructure defined in code (Terraform or Bicep)?
- CI/CD pipelines for every deployable component?
- Runbooks for common operational tasks?
- On-call rotation with documented escalation paths?
- Change management process for production changes?

**Performance Efficiency:**
- Load testing performed against production-like environment?
- Caching strategy defined (Azure Cache for Redis, CDN, application-level)?
- Database query performance validated under expected load?
- Auto-scaling triggers tested — do they scale fast enough?
- P95/P99 latency targets defined and monitored?

### Review Triggers

Architecture reviews happen when:
- **New workload:** Before any new solution reaches production
- **Significant change:** New service added, architecture pattern changed, region added
- **Incident:** Post-incident review reveals architectural weakness
- **Annual review:** Every production workload reviewed at least annually — technology and requirements evolve

### Review Output

Every review produces:

| Finding | Severity | Recommendation | Owner | Timeline |
|---------|----------|---------------|-------|----------|
| No diagnostic settings on Service Bus | High | Add diagnostic settings to Log Analytics | Monitoring Engineer | Sprint 24 |
| Standing Contributor role on prod RG | Critical | Move to PIM with time-limited activation | Identity Specialist | Immediate |
| No load testing performed | Medium | Run load test before next release | Compute Engineer | Sprint 25 |

Severity levels:
- **Critical:** Security gap or reliability risk — fix before next deployment
- **High:** Significant gap — fix within current sprint
- **Medium:** Improvement needed — schedule within next two sprints
- **Low:** Best practice recommendation — add to backlog

### The Specialty Lead's Role

The Specialty Lead ensures reviews are comprehensive, not just deep in one domain. A review that validates compute performance but ignores database reliability isn't a review — it's a spot check. Every pillar, every domain, every cross-cutting concern.
