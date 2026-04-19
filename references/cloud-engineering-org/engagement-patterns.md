## Common Engagement Patterns

### "Migrate us to Azure"
1. **Scoping**: Greenfield vs brownfield, inventory of workloads, compliance requirements
2. **Identity & Productivity**: Hybrid identity setup (Entra Connect / Cloud Sync), Conditional Access baseline
3. **Platform Engineering**: Landing zones, subscription structure, networking foundation
4. **Azure Specialty**: Workload assessment, technology selection per workload, migration strategy (rehost/refactor/rearchitect)
5. **DevOps**: CI/CD pipelines for migrated workloads, IaC for new infrastructure
6. **Shared Services**: Security validates throughout, cost models projected spend, docs capture everything

### "Build a new application on Azure"
1. **Scoping**: Requirements, architecture constraints, compliance, team capabilities
2. **Platform Engineering**: Landing zone readiness (may already exist)
3. **Azure Specialty**: Architecture design — compute, database, storage, networking, monitoring, integration
4. **DevOps**: CI/CD pipeline, IaC modules, deployment strategy
5. **Identity & Productivity**: App registration, managed identity, user authentication flow
6. **Shared Services**: Security review architecture, cost model the projected spend, docs capture ADRs

### "Modernize our M365 environment"
1. **Scoping**: Current state, licensing, compliance, user adoption challenges
2. **Identity & Productivity**: Identity foundation (Conditional Access, zero trust), M365 governance (Purview, Intune), licensing optimization, adoption strategy
3. **Azure Specialty**: May need Azure integration (hybrid scenarios, data platforms)
4. **DevOps**: May need automation for M365 admin tasks
5. **Shared Services**: Security validates identity and compliance posture, cost reviews licensing, docs capture policies

### "Our Azure costs are out of control"
1. **Scoping**: Current spend, budget, what's been tried
2. **Cost Optimization**: Immediate — identify waste, right-sizing opportunities, orphaned resources
3. **Azure Specialty**: Architecture review — are we using the right services and SKUs?
4. **Platform Engineering**: Governance review — are policies preventing cost sprawl?
5. **DevOps**: Pipeline review — are we deploying efficiently? Are dev environments being cleaned up?
6. **Shared Services**: Documentation captures optimization plan, security ensures cost cuts don't create security gaps
