# Deployment Types & Token Economics

## Deployment Types Overview

### Standard (Pay-As-You-Go)
- **Billing**: Per-token pricing (input tokens + output tokens)
- **Commitment**: None — pay only for what you use
- **Latency**: Variable, best-effort
- **Best for**: Development, testing, variable/unpredictable workloads, getting started
- **Limitations**: No latency guarantees, subject to regional capacity constraints
- **Regional**: Traffic stays in the deployed region

### Provisioned Throughput Units (PTU)
- **Billing**: Reserved capacity billed hourly (whether used or not)
- **Commitment**: Monthly reservation recommended for cost savings
- **Latency**: Predictable, consistent — guaranteed throughput
- **Best for**: Production workloads with consistent high volume, latency-sensitive applications
- **PTU sizing**: 1 PTU capacity varies by model (use Azure AI Foundry Capacity Calculator)
- **Minimum**: Varies by model, typically starts at 50-100 PTU
- **Key rule**: Buy reservations for production PTU — unreserved PTU billed at on-demand rates

### Global Standard
- **Billing**: Per-token, same as Standard
- **Routing**: Traffic routes to any Azure region with available capacity
- **Latency**: Often lower than regional due to geographic distribution
- **Best for**: Global applications, maximizing availability, highest default quota
- **Data processing**: Data may be processed in any Azure region

### Global Provisioned
- **Billing**: Reserved PTU with global routing
- **Best for**: Production workloads needing both guaranteed throughput and global availability
- **Data processing**: Data may be processed in any Azure region

### Data Zone Standard
- **Billing**: Per-token
- **Routing**: Traffic stays within geographic boundary (US or EU)
- **Best for**: Data residency requirements where you need geographic compliance but not single-region
- **Boundaries**: US data zone, EU data zone

### Data Zone Provisioned
- **Billing**: Reserved PTU within data zone
- **Best for**: Data residency + predictable throughput requirements

### Global Batch
- **Billing**: 50% discount off Standard pricing
- **Processing**: Asynchronous — submit jobs, results within 24 hours
- **Best for**: Bulk processing, embedding generation, evaluations, classification at scale
- **Limitations**: No real-time response, 24-hour SLA for completion

## Deployment Type Comparison Table

| Type | Data Processing | Billing | Latency | Best For |
|------|----------------|---------|---------|----------|
| Standard | Single region | Per-token | Variable | Dev/test, variable workloads |
| Provisioned (PTU) | Single region | Reserved hourly | Predictable | Regional production |
| Global Standard | Any Azure region | Per-token | Low (geo-routed) | Global apps, highest quota |
| Global Provisioned | Any Azure region | Reserved PTU | Predictable + global | Global production |
| Data Zone Standard | Within US/EU zone | Per-token | Variable | Data residency compliance |
| Data Zone Provisioned | Within US/EU zone | Reserved PTU | Predictable | Data residency + throughput |
| Global Batch | Any Azure region | 50% off Standard | 24-hour SLA | Bulk async processing |

## PTU vs Pay-As-You-Go Decision Framework

```
What's your usage pattern?
├─ Predictable, steady throughput?
│  ├─ >100K tokens/minute sustained? → PTU with reservation
│  ├─ 10K-100K tokens/minute? → PTU if latency matters, Standard if not
│  └─ <10K tokens/minute? → Standard (PTU minimum is too high)
├─ Spiky or unpredictable?
│  ├─ Need low latency during spikes? → PTU sized for peak + spillover to Standard
│  └─ Can tolerate variable latency? → Standard
├─ Batch/async workloads?
│  └─ Global Batch (50% discount, 24-hour completion)
└─ Dev/test/experimentation?
   └─ Standard. Never PTU for dev environments.
```

## Cost Optimization Strategies

### Start with Standard, Graduate to PTU
1. Deploy on Standard (pay-per-token) initially
2. Measure actual token consumption over 2-4 weeks
3. Calculate break-even: PTU becomes cheaper at ~60-70% sustained utilization
4. Switch to PTU when usage is predictable and exceeds break-even point
5. Always buy reservations for production PTU (significant savings over on-demand)

### Use Batch for Non-Interactive Workloads
- Embedding generation for RAG indexes → Global Batch
- Bulk classification or extraction → Global Batch
- Evaluation runs → Global Batch
- Any workload that doesn't need real-time response → Global Batch (50% savings)

### Right-Size Your Models
- GPT-4.1-nano for classification, extraction, routing (cheapest)
- GPT-4.1-mini for most production workloads (10-20x cheaper than flagship)
- GPT-4.1 or o3 only for tasks that demonstrably fail on smaller models
- Benchmark quality on your actual tasks before choosing larger models

### Spillover Configuration
- Enable spillover to route overflow traffic from PTU to Standard
- Prevents 429 errors without over-provisioning PTU capacity
- Only pay Standard rates for overflow tokens

## Token Cost Reference (Approximate)

| Model | Input (per 1M tokens) | Output (per 1M tokens) |
|-------|----------------------|----------------------|
| GPT-4o | ~$2.50 | ~$10.00 |
| GPT-4.1 | ~$2.00 | ~$8.00 |
| GPT-4.1-mini | ~$0.40 | ~$1.60 |
| GPT-4.1-nano | ~$0.10 | ~$0.40 |
| o3 | ~$10.00 | ~$40.00 |
| o4-mini | ~$1.10 | ~$4.40 |
| text-embedding-3-small | ~$0.02 | N/A |
| text-embedding-3-large | ~$0.13 | N/A |

*Prices are approximate and vary by deployment type and region. Check Azure pricing page for current rates.*

## PTU Sizing Guidelines

- Use the Azure AI Foundry Capacity Calculator (`ai.azure.com` → Quotas → Provisioned tab)
- Benchmark with real traffic patterns, not synthetic loads
- Monitor `Provisioned-managed utilization V2` metric in Azure Monitor
- Target 60-80% utilization — headroom prevents 429 errors at peaks
- PTU quota is per-subscription, per-region, per-deployment-type
- At 100% utilization, requests start getting throttled (429 responses)

## Common Cost Traps

- **Unreserved PTU**: Billed hourly whether used or not. 300 PTU deployed 24/7 without a reservation = very expensive
- **Embedding volume**: text-embedding-3-small is cheap per-call, but 100M documents × 500 tokens = 50B tokens = ~$1,000 just for initial embedding
- **Reasoning tokens**: o3/o4-mini generate internal reasoning tokens that are billed as output tokens — total cost can be 3-5x visible output
- **Over-provisioning**: PTU sized for peak but running at 20% utilization most of the time — Standard would be cheaper
