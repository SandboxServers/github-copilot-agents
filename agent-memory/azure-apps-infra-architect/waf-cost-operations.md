# Well-Architected: Cost & Operations

## Cost Optimization
### Right-Sizing
- Dev/test: Use Azure Dev/Test pricing (significant discounts)
- Non-production: Smallest viable SKU. Scale up only if needed.
- Production: Start small, monitor, scale based on metrics not guesses.

### Commitment Discounts
| Type | Discount | Commitment | Best For |
|------|----------|------------|----------|
| Reserved Instances | Up to 72% | 1-3 year | Steady-state VMs, databases |
| Savings Plans | Up to 65% | 1-3 year | Variable compute across services |
| Spot VMs | Up to 90% | None (evictable) | Batch processing, CI/CD agents |
| Dev/Test Pricing | Up to 55% | Visual Studio subscription | Non-production environments |

### Common Cost Traps
- Over-provisioned App Service plans (P3v3 running a blog)
- Premium storage on dev databases
- Orphaned public IPs, disks, NICs after VM deletion
- Always-on premium functions that barely execute
- Unused Application Gateway / Front Door instances
- Paying for unused reserved capacity

### Auto-Shutdown
```bash
# Auto-shutdown dev VMs at 7PM
az vm auto-shutdown -g rg -n devvm --time 1900 --timezone "Eastern Standard Time"
```

## Operational Excellence
### IaC Everything
No manual portal changes. Bicep or Terraform for all infrastructure. Deployment stacks for lifecycle.

### Monitoring Stack
Application Insights (app telemetry) + Log Analytics (infrastructure logs) + Azure Monitor (metrics + alerts). Diagnostic settings on every resource.

### Deployment Safety
1. What-if / plan before apply
2. Deploy to dev → staging → prod (environment promotion)
3. Feature flags for gradual rollout
4. Rollback plan documented before every production change

### Performance Efficiency
- Redis Cache for session state and hot data
- CDN (Front Door) for static content
- Connection pooling for databases
- Async processing with Service Bus for long operations
- Database indexing and query optimization
