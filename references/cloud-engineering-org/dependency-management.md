## Dependency Management

### Common Dependency Chains

```
Identity decisions → gate everything
  ├─ Platform (landing zones) → need RBAC model
  ├─ Specialty (all services) → need managed identity strategy
  ├─ DevOps (pipelines) → need service principals / workload identity
  └─ M365 (everything) → need Entra foundation

Platform (landing zones) → gate infrastructure
  ├─ Specialty (all services) → need subscriptions, networking, governance
  └─ DevOps (deployment targets) → need environments to deploy to

Network topology → gate service deployment
  ├─ Database (private endpoints)
  ├─ Compute (VNet integration)
  ├─ Storage (private endpoints)
  └─ Integration (service endpoints)

Architecture decisions → gate IaC
  ├─ DevOps can't write Terraform until Specialty decides what to build
  └─ Platform can't create modules until architecture is defined
```

### When to Parallelize vs Serialize

**Parallelize when**:
- Workstreams have no data or infrastructure dependencies
- Different divisions are working in isolated domains
- Phase 2 foundation work is complete and multiple build tracks can proceed

**Serialize when**:
- One division's output is another's input (architecture → IaC → pipeline)
- Identity decisions haven't been made yet (everything waits)
- Network topology is still being designed (service deployment waits)
- Security has flagged a blocker (everything in that track waits)

### Cross-Division Handoff Protocol

When work passes from one division to another:
1. **Clear deliverable**: What exactly is being handed off? What state is it in?
2. **Acceptance criteria**: What does the receiving division need to proceed?
3. **Documentation**: Decisions made, alternatives considered, constraints discovered
4. **Contact point**: Who to ask when questions arise during the next phase
