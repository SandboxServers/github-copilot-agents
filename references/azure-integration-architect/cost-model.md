## Cost Model

### Logic Apps
| Model | Billing | Best For |
|---|---|---|
| **Consumption** | Per trigger + per action execution | Low volume, simple flows |
| **Standard** | Fixed WS1/WS2/WS3 plan (like App Service) | High volume, multiple workflows, VNet |

- **Consumption cost trap**: loops are expensive — each action × each iteration × retries counts as an execution
- **Standard**: predictable monthly cost regardless of execution count. Multiple workflows share the plan.

### Service Bus
| Tier | Billing |
|---|---|
| **Basic** | Per operation (messaging operations) |
| **Standard** | Base + per operation + relay hours |
| **Premium** | Per messaging unit (MU) — fixed monthly (~$668/MU) |

- Premium MU sizing: start with 1 MU, scale based on throughput needs (~4 MB/s per MU)
- Auto-scale available in Premium to handle traffic spikes without over-provisioning

### Event Grid
- Per operation: first 100K operations/month free, then per million operations
- System topics from Azure resources: free to create, pay for operations
- Custom topics: per operation
- Very cost-effective for event-driven architectures at scale

### APIM
| Tier | Billing |
|---|---|
| **Consumption** | Per 10,000 calls (first 1M free/month) |
| **Classic (Dev/Basic/Standard/Premium)** | Per unit per hour |
| **V2 (Basic v2/Standard v2/Premium v2)** | Per unit per hour + overage per 10K calls |

- **Classic Premium**: ~$2,794/unit/month — expensive, but required for multi-region, full VNet injection, AZ
- **Standard v2**: much more cost-effective for VNet integration scenarios
- **Consumption**: cheapest entry point, but cold start and limited features
