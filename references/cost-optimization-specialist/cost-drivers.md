## Cost Drivers by Service

### Compute
- **VM size and series** — most impactful decision. Right-size based on actual CPU/memory/network usage.
- **Running hours** — dev/test VMs running 24/7 vs business hours = 3x cost difference
- **Spot VMs** — up to 90% savings for interruptible workloads (batch, dev/test, CI/CD agents)
- **Autoscale** — scale in when demand drops, don't over-provision for peak

### Storage
- **Access tier** — Hot vs Cool vs Cold vs Archive. Wrong tier = massive overspend.
- **Transactions** — storage account with millions of transactions costs more than the data stored
- **Redundancy** — LRS vs ZRS vs GRS vs GZRS. Don't pay for geo-redundancy on dev data.
- **Lifecycle policies** — automatically tier down or delete data based on age
- **Reserved capacity** — 1-year reservation for stable storage workloads

### Cosmos DB
- **Request Units (RUs)** — the #1 cost trap. Every read/write/query consumes RUs.
- **Provisioned vs autoscale vs serverless** — choose based on traffic patterns
- **Partition key design** — bad keys cause hot partitions and force higher RU provisioning
- **Indexing policy** — default indexes everything. Exclude unnecessary paths to reduce RU cost.
- **Multi-region** — storage and RU cost multiplied per region. Only replicate what you need.
- **Throughput mode** — Advisor recommends switching between manual and autoscale based on patterns

### Networking
- **Egress** — data leaving Azure costs money. Cross-region and internet egress add up.
- **ExpressRoute** — circuit cost + Premium add-on + metered vs unlimited data
- **Azure Firewall** — ~$900/month base. Firewall Manager for multi-hub can be expensive.
- **Application Gateway / Front Door** — WAF adds cost, but cheaper than a breach
- **Private endpoints** — per-endpoint cost. Many endpoints across many services adds up.

### App Service
- **Plan tier** — Free/Shared for dev, Basic for staging, Standard/Premium for prod. Don't run dev on Premium.
- **Unused slots** — deployment slots cost money even when not receiving traffic
- **Always On** — required for production but costs more than Basic tier

### AKS
- **Node pool sizing** — right-size nodes, use cluster autoscaler, fine-tune scale-down profile
- **Spot node pools** — for interruptible workloads (batch, CI agents)
- **Vertical Pod Autoscaler** — right-size pod resource requests/limits
- **Cost Analysis (native)** — built-in AKS cost analysis by Kubernetes construct (namespace, deployment)

### Monitoring (Log Analytics / App Insights)
- **Data ingestion** — the primary cost driver. Noisy resources can ingest GBs/day unnecessarily.
- **Commitment tiers** — 100GB/day+ gets discounts. Consider dedicated cluster for multi-workspace.
- **Data retention** — default 30 days included. Extended retention costs extra.
- **Sampling** — Application Insights adaptive sampling reduces telemetry volume without losing insights
