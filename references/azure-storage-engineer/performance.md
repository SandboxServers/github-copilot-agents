## Performance Characteristics

### Storage Account Limits (Standard GPv2)

| Metric | Limit |
|--------|-------|
| Max ingress per account | 10 Gbps (can request increase) |
| Max egress per account | 50 Gbps (can request increase) |
| Max request rate per account | 20,000 requests/sec |
| Max IOPS (per account) | 20,000 |
| Max blob size (block blob) | 190.7 TiB |
| Max single put (block blob) | 5 GiB |
| Max block count | 50,000 blocks |
| Max block size | 4 GiB |

### Premium Block Blob Storage
- For high-transaction workloads
- SSD-backed, consistent low latency
- No access tiers (Hot only)
- Higher cost per GB, lower cost per transaction
- Use when: IoT telemetry, real-time analytics, AI/ML training data, interactive workloads

### Performance Troubleshooting Checklist
1. **Throttling (429/503)?** → Check account-level IOPS/throughput limits → consider partitioning across accounts
2. **High latency?** → Check blob size (large blobs = higher latency), network path, storage tier, region proximity
3. **Slow uploads?** → Use parallel upload, increase block size, use AzCopy with concurrency tuning
4. **Slow large blob reads?** → Use range reads, parallel download, consider CDN for static content
5. **Lifecycle policies slow?** → Narrow scope with prefixes/tags, avoid small file tiering
6. **Hot partition?** → Blob names starting with same prefix → Azure auto-partitions but initial hotspots possible → use hash prefix
