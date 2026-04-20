# Azure Compute Best Practices

Proven patterns for running production compute workloads on Azure — covering sizing, availability, security, scaling, and cost.

## Right-Sizing

- **Monitor before committing:** Collect actual CPU/memory/disk metrics for 2+ weeks via Azure Monitor before selecting final VM sizes
- **Use Azure Advisor right-sizing recommendations:** Advisor identifies VMs consistently below 5% CPU or with oversized SKUs
- **Apply comfort factor:** When migrating, use 1.2-1.5x multiplier on observed utilization to account for spikes — not 3x
- **Review quarterly:** Workload patterns shift — schedule recurring right-sizing reviews using Advisor + Cost Management
- **Use Azure Migrate for lift-and-shift:** Performance-based assessments use 95th-percentile utilization to recommend optimal sizes

## Availability & Resilience

- **Use availability zones for production workloads:** Distributes VMs across physically separate datacenters — protects against zone-level failures
- **Premium SSD or Ultra Disk for single-VM SLA:** Single VMs require all premium-tier managed disks for the highest uptime SLA
- **Use zone-redundant storage (ZRS) disks** when available — data survives zone outages, supports force-detach for recovery
- **Implement health probes on all load-balanced endpoints:** Both Application Gateway and Azure Load Balancer rely on probes to route traffic to healthy instances
- **Separate application tiers into distinct availability sets/zones:** Don't co-locate web tier and database tier VMs in the same failure domain
- **Enable automatic instance repairs on VMSS:** Detects and replaces unhealthy instances based on health extension or load balancer probe status

## Auto-Scaling

- **Define scale rules based on actual metrics, not guesses:** Use CPU, memory, request queue depth, or custom metrics — not arbitrary schedules
- **Scale out aggressively, scale in conservatively:** Scale-out cooldown of 5 minutes; scale-in cooldown of 10+ minutes to avoid flapping
- **Always set min and max instance counts:** Prevent runaway scaling (cost explosion) and under-provisioning (outage)
- **AKS: enable cluster autoscaler per node pool** with appropriate min/max — allows scale-to-zero for GPU and spot pools
- **Use KEDA for event-driven scaling in AKS/Container Apps:** Scales based on queue depth, HTTP requests, cron, or custom metrics
- **Consider virtual nodes (ACI) for AKS burst scenarios:** Near-instant pod scheduling without waiting for node provisioning

## AKS Best Practices

- **Separate system and user node pools:** System pool runs kube-system pods (CoreDNS, metrics-server); user pools run application workloads — prevents resource contention
- **Use ephemeral OS disks for AKS nodes:** Faster boot times, lower latency, no storage cost — ideal since node OS state is disposable
- **Set pod resource requests and limits on all workloads:** Enables proper scheduling and prevents noisy-neighbor issues
- **Use pod disruption budgets (PDBs):** Ensures minimum availability during node drains, upgrades, and autoscaler scale-in
- **Enable LocalDNS for DNS-heavy workloads:** Per-node DNS proxy reduces latency and avoids conntrack pressure
- **Upgrade Kubernetes regularly:** Newer versions contain performance improvements, security patches, and reduced API throttling

## Container Apps Best Practices

- **Use revision management and traffic splitting:** Deploy new versions as revisions; gradually shift traffic (canary/blue-green) before promoting
- **Keep container images small:** Smaller images = faster cold starts on Consumption plan; target <500 MB
- **Use Dapr for service-to-service communication:** Built-in service invocation, pub/sub, state management without custom plumbing
- **Choose Dedicated (workload profiles) for predictable latency:** Consumption plan has cold starts; Dedicated provides pre-allocated compute

## Security

- **Use managed identity on ALL compute resources:** VMs, VMSS, AKS, App Service, Functions, Container Apps — eliminate stored credentials
- **Match managed identity region to compute region:** Use one user-assigned identity per region for isolation
- **Enable Microsoft Defender for Cloud on compute:** Detects threats, vulnerabilities, and misconfigurations
- **Use private endpoints for dependent services:** Ensure compute resources access storage, databases, and Key Vault over private network

## Cost Optimization

- **Tag compute resources with cost-center and environment:** Enables chargeback, showback, and automated enforcement of dev/test policies
- **Reserved capacity for stable baseline, on-demand for peaks:** Buy 1-year or 3-year RIs/savings plans for predictable workloads; use pay-as-you-go for burst
- **Savings plans over RIs when flexibility matters:** Savings plans apply across VM families, regions, and compute services
- **Spot VMs for fault-tolerant workloads:** Implement checkpoint/restart logic for long-running jobs; use eviction-policy Delete with ephemeral OS disks
- **Auto-shutdown dev/test environments:** DevTest Labs or scheduled VMSS scaling to eliminate off-hours waste
- **Scale-to-zero where possible:** Functions Consumption, Container Apps Consumption, AKS spot node pools with autoscaler min=0

## Spot VM Best Practices

- **Implement checkpoint/restart logic for long-running jobs:** Spot eviction gives ~30 seconds; save state to durable storage at regular intervals
- **Use multiple VM sizes in spot pool:** Increases likelihood of capacity allocation; use attribute-based VM selection in Compute Fleet
- **Use Delete eviction policy with ephemeral OS disks:** Avoids paying for dangling disks after eviction; enables faster redeployment
- **Monitor eviction rates by SKU and region:** Use spot placement score and eviction history to choose optimal configurations
- **Never run stateful production workloads on spot:** Spot is for batch, dev/test, CI/CD, and fault-tolerant processing only

## Monitoring & Observability

- **Enable VM Insights on all VMs and VMSS:** Provides process-level CPU/memory/disk maps without installing custom agents
- **Use Container Insights for AKS:** Node-level and pod-level metrics, log collection, live data view
- **Set up alerts on key thresholds:** CPU >80% sustained, memory >85%, disk queue depth >2 for 5+ minutes
- **Use Azure Monitor Workbooks for compute dashboards:** Combine metrics across VMs, VMSS, AKS, and App Service in single view
- **Track cost anomalies with Cost Management alerts:** Catch runaway scaling or forgotten dev environments early

## Operational Excellence

- **Use Infrastructure as Code (Bicep/Terraform) for all compute:** Reproducible deployments, drift detection, consistent environments
- **Enable automatic OS patching:** VMSS automatic OS upgrades, AKS node image auto-upgrade, App Service platform updates
- **Test disaster recovery regularly:** Use Azure Site Recovery for VM failover; test AKS cluster recreation from IaC
- **Document VM sizing rationale:** Record why each size was chosen — prevents "why is this so big?" questions 6 months later

## References

- [Architecture best practices for VMs and scale sets](https://learn.microsoft.com/azure/well-architected/service-guides/virtual-machines)
- [Best practices for AKS performance and scaling](https://learn.microsoft.com/azure/aks/best-practices-performance-scale)
- [FinOps best practices for compute](https://learn.microsoft.com/cloud-computing/finops/best-practices/compute)
- [Build workloads on spot virtual machines](https://learn.microsoft.com/azure/architecture/guide/spot/spot-eviction)
- [Azure Compute Fleet — attribute-based VM selection](https://learn.microsoft.com/azure/azure-compute-fleet/attribute-based-vm-selection)
