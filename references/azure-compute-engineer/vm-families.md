## VM Families Quick Reference

| Family | Type | Best For | Key Characteristic |
|--------|------|----------|-------------------|
| **A** | General purpose | Dev/test, entry-level | Budget-friendly, basic performance |
| **B** | General purpose (burstable) | Low-traffic web, dev/test, CI agents | CPU credit model — bursts above baseline, throttles when credits exhausted |
| **D** | General purpose | Production web/app servers, databases | Balanced CPU-to-memory, workhorse family |
| **E** | Memory optimized | Large DBs, SAP, in-memory analytics | High memory-to-CPU ratio (8:1 GiB:vCPU) |
| **F** | Compute optimized | Batch processing, gaming, analytics | High CPU-to-memory ratio, fast processors |
| **L** | Storage optimized | Big data, SQL/NoSQL DBs, data warehousing | High disk throughput, large local NVMe/SSD |
| **M** | Ultra memory optimized | SAP HANA, mega databases, HPC | Up to 4 TB RAM, massive in-memory workloads |
| **N** | GPU accelerated | AI/ML training, rendering, HPC | NVIDIA GPUs (A100, H100, T4, V100) |

### VM Naming Convention Decoder
```
Standard_D8as_v5
         │││││ └─ Version (v5 = latest generation)
         ││││└── Suffix: s = premium SSD capable
         │││└─── Suffix: a = AMD processor (no suffix = Intel)
         ││└──── vCPU count: 8
         │└───── Family: D = general purpose
         └────── Type prefix
```

### Common Suffixes
| Suffix | Meaning |
|--------|---------|
| `s` | Premium SSD capable |
| `a` | AMD processor |
| `d` | Local temp disk included |
| `i` | Isolated (dedicated host) |
| `l` | Low memory (relative to family) |
| `p` | ARM-based (Ampere) |
| `t` | Tiny memory (constrained) |

### B-Series Credit Model (Common Gotcha)
- B-series VMs accumulate CPU credits when idle (below baseline)
- Credits consumed when bursting above baseline
- When credits exhausted → throttled to baseline (e.g., B2s baseline = 40% of 2 vCPUs)
- **Gotcha**: Running sustained workloads on B-series → constant throttling → terrible performance
- **Use for**: Dev/test, CI agents, low-traffic web servers, jump boxes
- **Never use for**: Production databases, sustained compute, anything that needs consistent CPU
