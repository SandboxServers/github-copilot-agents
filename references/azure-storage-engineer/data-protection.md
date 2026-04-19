## Data Protection Feature Matrix

| Feature | Description | Cost Impact | Recommended Retention |
|---------|-------------|-------------|----------------------|
| **Container soft delete** | Recover deleted containers | Capacity during retention | 7+ days |
| **Blob soft delete** | Recover deleted blobs/versions | Capacity during retention | 7-14 days |
| **Blob versioning** | Auto-save on every write | New version per write → capacity growth | Enable + lifecycle rules for old versions |
| **Point-in-time restore** | Restore account to prior state | Requires versioning + soft delete + change feed | 1-7 days (longer = more cost) |
| **Immutability** | WORM compliance | No additional charge (just can't delete) | Per regulatory requirement |
| **Change feed** | Log of all blob changes | Small capacity + transaction cost | Required for point-in-time restore |
| **Snapshots** | Manual point-in-time capture | Billed for unique blocks only | Use lifecycle to clean up |

### Recommended Data Protection Configuration
```
Minimum (all production accounts):
├─ Container soft delete: 7 days
├─ Blob soft delete: 7 days
└─ Resource lock on storage account

Standard (most production workloads):
├─ Container soft delete: 14 days
├─ Blob soft delete: 14 days
├─ Blob versioning: Enabled
├─ Lifecycle rule: Delete versions > 90 days
└─ Resource lock on storage account

Maximum (compliance / critical data):
├─ Container soft delete: 30 days
├─ Blob soft delete: 30 days
├─ Blob versioning: Enabled
├─ Point-in-time restore: 7 days
├─ Change feed: Enabled
├─ Immutability policies (per container/version)
├─ Lifecycle rule: Delete versions > 90 days, tier snapshots
└─ Resource lock on storage account
```
