# Azure Cost Gotchas

Pricing surprises and hidden costs that catch teams off guard.
Knowing these up front prevents budget overruns and awkward conversations with finance.

## Commitment Discount Gotchas

### Reserved Instances
- **Use-it-or-lose-it:** Unused hours are forfeited — no rollover to next day
- **Exchange limitations:** Can exchange for same-type reservations but exchanges are being phased out (limited after Jan 2024 for new purchases)
- **Refund cap:** Maximum $50,000 USD in refunds per rolling 12-month window per billing profile
- **Wrong scope:** Instance-level reservations don't float across subscriptions — use shared scope for flexibility
- **Region-locked:** Reservations are pinned to a specific Azure region — migrating workloads strands the reservation

### Savings Plans
- **Broader flexibility but lower discount:** Covers multiple VM families, regions, and compute services — but typically 5–15% less savings than equivalent reservation
- **Hourly commitment is fixed:** You pay the committed amount regardless of usage — underutilization is pure waste
- **No refunds or exchanges:** Once purchased, a savings plan cannot be cancelled or exchanged
- **Discount priority:** Azure applies reservations first, then savings plans — plan your mix accordingly

## Storage Gotchas

### Egress and Operations Charges
- **Capacity is just the start:** Read/write/list operations are billed separately from stored GB
- **Egress is per-GB:** Data leaving Azure (internet or cross-region) is charged at $0.05–$0.12/GB depending on volume and region
- **Intra-region is free:** Traffic between services in the same region and availability zone costs nothing
- **Lifecycle policies save quietly:** Auto-tiering from Hot to Cool to Archive reduces capacity cost but increases access cost when data is retrieved

### Cosmos DB
- **RU consumption ≠ storage cost:** Request Units (throughput) and storage are billed independently
- **Provisioned throughput is always-on:** You pay for provisioned RU/s even during zero-traffic periods
- **Autoscale has a floor:** Minimum is 10% of max RU/s — setting max to 10,000 means you always pay for at least 1,000 RU/s
- **Multi-region writes multiply cost:** Each additional write region roughly doubles RU charges

## Networking Gotchas

### Azure Front Door
- **Per-request pricing:** Base charge per million requests adds up fast for high-traffic APIs
- **Data transfer is separate:** Egress from Front Door edge to client is billed on top of request fees
- **WAF rules add cost:** Each custom WAF rule and managed ruleset has its own per-request charge

### Private Endpoints
- **$0.01/hour per endpoint:** Sounds trivial — but 100 endpoints across 5 environments = ~$3,650/month
- **Data processing charges:** $0.01/GB processed through private endpoints, separate from endpoint hourly cost
- **Scale creep:** Microservice architectures with per-service private endpoints multiply fast

### Data Transfer
- **Cross-region egress:** Charged even within the same subscription — $0.02–$0.08/GB depending on region pair
- **Internet egress:** First 100 GB/month included with some services, then $0.05–$0.12/GB tiered pricing
- **VNet peering:** Global peering (cross-region) is charged; same-region peering is much cheaper but not free

## Compute Gotchas

### AKS
- **Control plane is free:** No charge for the Kubernetes API server
- **Everything else is not:** Node VMs, managed disks, load balancers, public IPs, egress — all billed normally
- **System node pools can't scale to zero:** At least one system node pool must have running nodes
- **Persistent volumes:** Azure Disk and Azure Files charges continue even when pods are not running

### Azure Functions (Consumption Plan)
- **First million executions free per month** — then $0.20 per million executions plus $0.000016/GB-s of memory
- **Cold start penalty is cost-free but latency-expensive:** No charge during idle, but startup latency affects SLAs
- **Durable Functions orchestrations:** Each activity, timer, and sub-orchestration counts as a separate execution

### Spot VMs
- **Real savings (60–90% discount)** but eviction can happen with 30 seconds notice
- **No SLA:** Not suitable for stateful workloads or anything requiring guaranteed uptime
- **Price varies by region and VM family:** Some combinations rarely get evicted, others are volatile
- **Eviction policy matters:** Deallocate (keeps disk, no compute charge) vs. Delete (gone entirely)

## Monitoring Gotchas

### Log Analytics
- **Ingestion is the main cost driver:** Not retention — data ingestion at $2.30–$3.50/GB depending on tier
- **Verbose diagnostic settings:** Enabling all categories on every resource can 10x monitoring costs
- **Basic Logs tier:** 60% cheaper ingestion but limited query capabilities — good for high-volume, low-query data
- **Commitment tiers:** 100 GB/day and above get volume discounts — analyze ingestion volume before choosing

## Sources

- [Plan and manage costs for Azure Storage](https://learn.microsoft.com/azure/storage/common/storage-plan-manage-costs)
- [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/)
- [Reservation discount overview](https://learn.microsoft.com/azure/cost-management-billing/reservations/save-compute-costs-reservations)
- [Savings plan for compute overview](https://learn.microsoft.com/azure/cost-management-billing/savings-plan/savings-plan-compute-overview)
- [Cost Optimization Workbook](https://learn.microsoft.com/azure/advisor/advisor-workbook-cost-optimization)
