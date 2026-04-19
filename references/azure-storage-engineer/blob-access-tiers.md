## Blob Storage Deep Dive

### Access Tiers — The Cost Math

| Tier | Storage Cost | Access Cost | Min Retention | Retrieval Latency | Use Case |
|------|-------------|-------------|---------------|-------------------|----------|
| **Hot** | Highest | Lowest | None | Milliseconds | Frequently accessed data |
| **Cool** | ~40-50% less than Hot | Higher per-op | 30 days (early delete penalty) | Milliseconds | Infrequently accessed (monthly) |
| **Cold** | ~60-65% less than Hot | Higher than Cool | 90 days (early delete penalty) | Milliseconds | Rarely accessed (quarterly) |
| **Archive** | ~90% less than Hot | Highest per-op + rehydration | 180 days (early delete penalty) | Hours (standard) to minutes (high priority) | Long-term retention, compliance |
| **Smart Tier** | Auto-transitions Hot→Cool→Cold | Small monitoring fee | None | Milliseconds (online tiers) | Unknown/variable access patterns |

### Smart Tier (New)
- Auto-moves data: Hot → Cool after 30 days idle → Cold after 90 days idle
- Auto-promotes back to Hot on access
- No tier transition charges, no early delete penalties, no rehydration costs
- Requires ZRS, GZRS, or RA-GZRS (not available on LRS/GRS)
- Only block blobs — not page or append blobs
- Great for "we don't know our access pattern" scenarios

### Early Delete Penalty Math
Moving a blob OUT of a tier before the minimum retention period charges you for the remaining days. Example: blob in Cool tier for 10 days, then deleted → you pay for 20 more days of Cool storage.

**Key insight**: Don't set lifecycle policies to move blobs to Cool if they might be accessed within 30 days. The transaction cost + early delete penalty can exceed Hot storage costs.

### The Break-Even Calculation
```
When does Cool tier save money over Hot?
─ Storage savings must exceed: (access cost increase × number of operations) + early delete risk
─ Rule of thumb: if accessed less than once per month → Cool
─ If accessed less than once per quarter → Cold
─ If accessed less than twice per year → Archive

When does moving small files cost MORE than leaving them in Hot?
─ Transaction costs dominate for small files
─ Moving 1 million 1KB files to Cool costs more in transactions than Hot storage savings
─ Rule: Don't tier files < 128KB unless they're truly dormant
```
