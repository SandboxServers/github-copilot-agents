# Deployment Types & Token Economics

## Deployment Type Comparison

| Type | SKU Code | Data Processing | Billing | Best For |
|------|----------|----------------|---------|----------|
| Global Standard | `GlobalStandard` | Any Azure region | Pay-per-token | General workloads, highest quota |
| Global Provisioned | `GlobalProvisionedManaged` | Any Azure region | Reserved PTU | Predictable high-throughput production |
| Global Batch | `GlobalBatch` | Any Azure region | 50% discount, 24-hr | Large async jobs (evals, embeddings) |
| Data Zone Standard | `DataZoneStandard` | Within data zone (EU/US) | Pay-per-token | Data residency compliance |
| Data Zone Provisioned | `DataZoneProvisionedManaged` | Within data zone | Reserved PTU | Data zone + predictable throughput |
| Standard (Regional) | `Standard` | Single region | Pay-per-token | Strict regional compliance |
| Regional Provisioned | `ProvisionedManaged` | Single region | Reserved PTU | Regional compliance + throughput |

## PTU vs Pay-As-You-Go Decision Framework

```
What's your usage pattern?
├─ Predictable, steady throughput?
│  ├─ >100K tokens/minute sustained? → PTU with reservation (significant savings)
│  ├─ 10K-100K tokens/minute? → PTU if latency-sensitive, otherwise Standard
│  └─ <10K tokens/minute? → Standard (PAYG). PTU minimum is too high.
├─ Spiky or unpredictable?
│  ├─ Need low latency during spikes? → PTU (sized for peak) + spillover to Standard
│  └─ Can tolerate variable latency? → Standard (PAYG)
├─ Batch/async workloads?
│  └─ Global Batch (50% discount, results within 24 hours)
└─ Dev/test/experimentation?
   └─ Standard (PAYG). Never PTU for dev.
```

## Cost Trap Prevention

- **The $50K surprise**: PTU is billed hourly whether you use it or not. 300 PTU deployed 24/7 without a reservation = expensive. Always buy reservations for production PTU.
- **Spillover**: Optional capability that routes overflow traffic from PTU to Standard. Prevents 429s without over-provisioning PTU. Enable it.
- **Token math**: GPT-4o at ~$2.50/M input, ~$10/M output (Standard). At 1M tokens/day output = ~$300/day = ~$9K/month. Compare to PTU cost at your volume.
- **Embedding costs**: text-embedding-3-small is ~$0.02/M tokens. Cheap per-call but volume kills — 100M documents × 500 tokens = 50B tokens = $1,000 just for initial embedding. Plan for it.
- **Batch discount**: If you can tolerate 24-hr latency, Global Batch is 50% off Standard pricing. Use for evals, bulk classification, embedding generation.
- **Model selection for cost**: GPT-4.1-mini and GPT-4o-mini are 10-20x cheaper than full models. Use them for classification, extraction, simple chat. Reserve full models for complex reasoning.

## PTU Sizing Guidelines

- Use the Azure AI Foundry Capacity Calculator (ai.azure.com → Quotas → Provisioned tab)
- Benchmark with real traffic patterns, not synthetic
- Monitor `Provisioned-managed utilization V2` metric in Azure Monitor
- Target 60-80% utilization for headroom. 100% = 429 errors
- PTU quota is per-subscription, per-region, per-deployment-type (Global/Data Zone/Regional)
- Existing PTU reservations automatically apply to new Foundry Models (DeepSeek, Llama, etc.)
