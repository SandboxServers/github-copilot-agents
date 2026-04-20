# Storage Performance

Optimizing Azure Storage for throughput, latency, and scalability.

## Scalability Targets

| Metric | Standard GPv2 | Premium Block Blob |
|--------|--------------|--------------------|
| Max ingress per account | 10 Gbps (increasable) | Higher |
| Max egress per account | 50 Gbps (increasable) | Higher |
| Max request rate | 20,000 requests/sec | Higher IOPS |
| Max IOPS | 20,000 | SSD-backed |
| Per-blob request limit | 500 requests/sec | 500 requests/sec |
| Max block blob size | 190.7 TiB | 190.7 TiB |
| Max single put | 5 GiB | 5 GiB |
| Max block size | 4000 MiB | 4000 MiB |

## Partitioning Strategy

Blob names are partition keys. Poor naming creates hotspots.

| Pattern | Problem | Solution |
|---------|---------|----------|
| Sequential prefix (timestamp) | All writes hit same partition | Hash prefix or GUID prefix |
| Same container, same prefix | Partition hotspot, throttling | Spread across prefixes |
| Single blob high-read | 500 RPS limit per blob | CDN for static, multiple copies for dynamic |

## Block Blob Upload Optimization

- **Large blocks** for throughput: Up to 4000 MiB per block
- **Put Block List**: Assemble large blobs from parallel block uploads
- **Parallel upload**: SDK handles automatically with configurable concurrency
- **Staging**: Upload blocks in parallel, then commit with Put Block List

## AzCopy Best Practices

- Best tool for bulk transfer and migration
- Parallel transfers with automatic concurrency
- Resume capability for interrupted transfers
- Benchmark mode for testing throughput
- Use `--cap-mbps` to limit bandwidth impact
- Sync mode for incremental updates

## Client-Side Optimization

- Use latest SDK (parallel operations built-in)
- Connection pooling (reuse HttpClient instances)
- Retry with exponential backoff (SDK default)
- Set appropriate timeout values
- Use streaming for large blob reads (avoid loading into memory)

## Premium Storage

| Type | Backed By | Use Case |
|------|-----------|----------|
| **Premium block blobs** | SSD | High-transaction, consistent low latency |
| **Premium file shares** | SSD | Provisioned IOPS file workloads |
| **Premium page blobs** | SSD | VM disks (prefer Managed Disks) |

Use for: IoT telemetry, real-time analytics, AI/ML training data, latency-sensitive workloads.

## CDN Integration

- Azure CDN or Front Door for global content delivery
- Cache blob content at edge locations
- Reduce latency for static content serving
- Custom domain and HTTPS support
- Purge and preload capabilities

## Troubleshooting Checklist

1. **Throttling (429/503)?** → Check account IOPS/throughput limits → partition across accounts
2. **High latency?** → Check blob size, network path, tier, region proximity
3. **Slow uploads?** → Parallel upload, increase block size, AzCopy concurrency
4. **Slow large reads?** → Range reads, parallel download, CDN for static
5. **Lifecycle policies slow?** → Narrow scope with prefixes/tags
6. **Hot partition?** → Hash prefix blob names for parallel access

## Related References

- [storage-selection.md](storage-selection.md) — Premium vs Standard selection
- [blob-access-tiers.md](blob-access-tiers.md) — Tier impact on access latency
- [anti-patterns.md](anti-patterns.md) — Sequential naming anti-pattern
