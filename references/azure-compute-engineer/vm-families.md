# VM Families & Sizing

## VM Family Reference

| Series | Type | Use Case | vCPU Range | Memory Range | Key Notes |
|--------|------|----------|-----------|--------------|-----------|
| **A-series** | Entry-level general purpose | Dev/test, low-traffic web | 1-8 | 0.75-56 GiB | Budget-friendly, no premium storage, not for production |
| **B-series** | Burstable general purpose | Dev/test, CI agents, light workloads | 1-32 | 0.5-128 GiB | CPU credit model — bursts above baseline, throttles when exhausted |
| **D-series (Dv5, Ddsv5, Dasv5, Dpsv5)** | General purpose | Production web/app servers, databases | 2-96 | 8-384 GiB | Workhorse family. Balanced CPU:memory. Ddsv5 includes local temp disk |
| **E-series (Ev5, Edsv5, Easv5)** | Memory-optimized | SAP, large caches, in-memory analytics | 2-104 | 16-672 GiB | 8:1 GiB:vCPU ratio. SAP HANA certified (some SKUs) |
| **F-series (Fsv2)** | Compute-optimized | Batch processing, gaming, simulations | 2-72 | 4-144 GiB | High CPU:memory ratio, fastest per-core performance |
| **L-series (Lsv3, Lasv3)** | Storage-optimized | Big data, SQL/NoSQL, data warehousing | 8-80 | 64-640 GiB | High disk throughput, large local NVMe SSD |
| **M-series (Mv2, Msv2)** | Ultra memory-optimized | SAP HANA, mega databases | 8-416 | 218 GiB-12 TiB | Up to 12 TiB RAM on largest SKUs. SAP HANA production certified |
| **N-series (NC, ND, NV)** | GPU-accelerated | AI/ML, deep learning, rendering | 6-96 | 56-900 GiB | NC: compute (T4, A100). ND: deep learning (A100, H100). NV: visualization |

## VM Naming Convention

```
Standard_D8as_v5
         │││││ └─ Version (v5 = latest generation, always prefer latest)
         ││││└── Suffix: s = premium SSD capable
         │││└─── Suffix: a = AMD processor (no suffix = Intel)
         ││└──── vCPU count: 8
         │└───── Family: D = general purpose
         └────── Type prefix: Standard
```

## Common Suffixes

| Suffix | Meaning | Example |
|--------|---------|---------|
| `s` | Premium SSD capable | D4s_v5 |
| `a` | AMD processor (typically 5-10% cheaper than Intel) | D4as_v5 |
| `d` | Local temp disk included (NVMe or SSD) | D4ds_v5 |
| `i` | Isolated — dedicated physical host, no co-tenancy | E64is_v5 |
| `l` | Low memory relative to family | D4ls_v5 |
| `p` | ARM-based (Ampere Altra, best price-performance) | D4ps_v5 |
| `t` | Tiny/constrained memory | E4-2ts_v5 |
| `b` | Block storage performance (L-series) | L8bs_v3 |
| `c` | Confidential computing | DC4s_v3 |

## B-Series CPU Credit Model

B-series VMs accumulate CPU credits when running below their baseline performance:

| SKU | vCPUs | Baseline CPU | Credits/hour | Max Banked | RAM | Monthly Cost (approx) |
|-----|-------|-------------|-------------|-----------|-----|----------------------|
| B1s | 1 | 10% | 6 | 144 | 1 GiB | ~$8 |
| B2s | 2 | 40% | 24 | 576 | 4 GiB | ~$30 |
| B2ms | 2 | 60% | 36 | 864 | 8 GiB | ~$60 |
| B4ms | 4 | 90% | 54 | 1296 | 16 GiB | ~$120 |
| B8ms | 8 | 135% | 81 | 1944 | 32 GiB | ~$240 |

**How credits work:**
- Below baseline → accumulate credits (up to max banked)
- Above baseline → consume credits
- Credits exhausted → throttled to baseline (severe performance degradation)
- B1s at 10% baseline means only 10% of one vCPU sustained — extremely limited

**Use B-series for:** Dev/test VMs, jump boxes, CI agents, low-traffic web servers, DNS servers
**Never use B-series for:** Production databases, sustained compute, latency-sensitive applications

## Sizing Recommendations by Workload

| Workload | Recommended Family | Starting SKU | Scale Signal |
|----------|-------------------|-------------|--------------|
| Web/API server | D-series (Dv5) | D2s_v5 (2 vCPU, 8 GiB) | CPU > 70% p95 |
| SQL Server | E-series (Ev5) | E4s_v5 (4 vCPU, 32 GiB) | Memory pressure, disk IOPS |
| In-memory cache (Redis) | E-series (Ev5) | E8s_v5 (8 vCPU, 64 GiB) | Memory utilization |
| Batch processing | F-series (Fsv2) | F4s_v2 (4 vCPU, 8 GiB) | Queue depth, job completion time |
| SAP HANA | M-series (Mv2) | M32ms (32 vCPU, 875 GiB) | SAP sizing worksheet |
| ML training | N-series (NC) | NC24ads_A100_v4 (24 vCPU, 1x A100) | GPU utilization, training time |
| ML inference | N-series (NC) | NC4as_T4_v3 (4 vCPU, 1x T4) | Request latency, GPU utilization |
| Dev/test | B-series | B2s (2 vCPU, 4 GiB) | Cost, not performance |
| Data warehouse | L-series (Lsv3) | L8s_v3 (8 vCPU, 64 GiB) | Disk throughput, IOPS |

## Right-Sizing Process

1. **Deploy with initial estimate** — use the table above as starting point
2. **Monitor for 2+ weeks** — collect CPU, memory, disk IOPS, network metrics
3. **Analyze p95 metrics, not averages** — average CPU of 30% may have p95 of 85%
4. **Check Azure Advisor** — automated recommendations for underutilized VMs (CPU < 5%)
5. **Right-size first, then commit** — never buy Reserved Instances before right-sizing
6. **Re-evaluate quarterly** — workload patterns change, new VM families launch

## Common Sizing Mistakes

- Using D-series for memory-heavy workloads → E-series has 2x memory per vCPU
- Choosing v3/v4 when v5 is available → newer generations are cheaper per unit of performance
- Ignoring AMD variants (Dasv5, Easv5) → typically 5-10% cheaper, equivalent performance
- Overlooking ARM-based (Dpsv5) → best price-performance for Linux workloads
- Running B-series in production → credit exhaustion causes unpredictable throttling
- Sizing for peak load with no autoscaling → VMSS with autoscale rules is better
- Picking the biggest VM "to be safe" → most VMs run at < 20% utilization
