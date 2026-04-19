## Lifecycle Management Policies

### How They Work
- Rule-based automation: move blobs between tiers or delete based on age, last access time, or creation time
- Process runs daily (can take up to 24 hours to start after prior run completes)
- Rules applied per-account, filtered by container prefix or blob index tags
- Can target: current versions, previous versions, snapshots

### Policy Design Patterns

```json
// Pattern 1: Standard tiering cascade
"rules": [
  { "if": { "lastModified > 30 days" }, "then": { "tierToCool" } },
  { "if": { "lastModified > 90 days" }, "then": { "tierToCold" } },
  { "if": { "lastModified > 365 days" }, "then": { "tierToArchive" } },
  { "if": { "lastModified > 2555 days (7 years)" }, "then": { "delete" } }
]

// Pattern 2: Version cleanup (critical for cost control with versioning enabled)
"rules": [
  { "if": { "version created > 30 days ago" }, "then": { "tierToCool" } },
  { "if": { "version created > 90 days ago" }, "then": { "delete" } }
]

// Pattern 3: Snapshot cleanup
"rules": [
  { "if": { "snapshot created > 30 days ago" }, "then": { "delete" } }
]
```

### Lifecycle + Data Protection Interaction (Critical Knowledge)

| Feature | Lifecycle Impact | Cost Trap |
|---------|-----------------|-----------|
| **Versioning enabled** | Every write creates a version → capacity explodes without lifecycle rules for old versions | MUST add version cleanup rules |
| **Soft delete enabled** | Deleted blobs still consume capacity during retention period | Factor retention period into cost estimates |
| **Point-in-time restore** | Requires versioning + soft delete + change feed → all add cost | Retention period has cascade effect on version/soft delete retention |
| **Snapshots** | Lifecycle can auto-delete old snapshots | Without cleanup rules, snapshots accumulate indefinitely |
| **Immutable storage** | Lifecycle CANNOT delete blobs under legal hold or unexpired retention | Immutability overrides lifecycle delete rules |
