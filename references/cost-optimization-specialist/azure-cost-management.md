## Azure Cost Management

### Key Tools
- **Cost Analysis** — built-in portal experience for exploring costs by service, resource group, tag, subscription
- **Budgets** — set spending limits with alerts at configurable thresholds
- **Cost Exports** — scheduled exports to storage accounts for external analysis (Power BI, Synapse, Excel)
- **Cost Allocation Rules** — redistribute shared service costs across subscriptions/RGs/tags
- **Anomaly Detection** — automatic identification of unexpected cost spikes or drops
- **Advisor Recommendations** — system-generated cost optimization recommendations based on usage patterns
- **FinOps Hubs** — scalable platform for advanced cost analytics via Fabric/Data Explorer (~$120/mo base + $10/mo per $1M monitored)

### Cost Analysis Best Practices
1. Use **management groups, subscriptions, and resource groups** as natural cost boundaries
2. **Filter and group by tags** — requires consistent tagging strategy
3. Enable **tag inheritance** in Cost Management — copies subscription/RG tags to child resources without changing actual tags
4. Use **custom dimensions** via the Cost Management Dimensions API for advanced grouping
5. Export to **Power BI or Synapse** for executive-friendly reporting
