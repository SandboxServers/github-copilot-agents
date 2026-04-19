## The 8 Rs of Migration Strategy

```
What should we do with this workload?
├─ No longer needed? → RETIRE (decommission)
├─ Can't move yet? → RETAIN (keep on-prem, manage with Arc)
├─ Move as-is? → REHOST (lift and shift to IaaS)
├─ Move to PaaS with minimal changes? → REPLATFORM (lift, tinker, shift)
├─ Optimize code for cloud? → REFACTOR (code changes, same behavior)
├─ Need cloud-native capabilities? → REARCHITECT (significant design changes)
├─ Replace with SaaS/AI? → REPLACE (adopt existing product)
└─ Start over? → REBUILD (new cloud-native application)
```

### Strategy Selection Guide

| Strategy | When to Use | Risk Level | Cost | Timeline |
|----------|------------|------------|------|----------|
| Retire | Workload obsolete, no dependencies | Low | Lowest | Immediate |
| Retain | Compliance blocker, not ready yet | Low | None (status quo) | Deferred |
| Rehost | Stable workload, datacenter exit needed | Low | Low | Weeks |
| Replatform | Want PaaS benefits, minimal code changes | Medium | Medium | Weeks-months |
| Refactor | Code optimization, reduce tech debt | Medium | Medium | Months |
| Rearchitect | Need microservices, event-driven, scaling | High | High | Months |
| Replace | SaaS/AI solution exists, custom code unnecessary | Medium | Varies | Months |
| Rebuild | Legacy beyond saving, greenfield opportunity | High | Highest | Months-quarters |

### Common Anti-pattern
**"Rehost everything first, modernize later"** — sounds pragmatic but often creates permanent technical debt. Organizations rarely go back and modernize. Better: use rehost for quick wins but plan modernization waves from the start.
