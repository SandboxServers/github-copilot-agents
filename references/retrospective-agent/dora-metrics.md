## DORA Metrics for Continuous Improvement

### The Four Key Metrics
| Metric | What It Measures | Why It Matters for Retrospectives |
|---|---|---|
| **Deployment Frequency** | How often code deploys to production | Low frequency = large batches = higher risk per deployment |
| **Lead Time for Changes** | Commit to production duration | Long lead times reveal process bottlenecks |
| **Change Failure Rate** | % of deployments causing failures | High rate reveals testing or review gaps |
| **Mean Time to Restore** | Time from failure to recovery | Long MTTR reveals detection or response gaps |

### Using DORA in Retrospectives
- Track these metrics sprint-over-sprint and engagement-over-engagement
- Identify trends: are we getting faster or slower? More reliable or less?
- Correlate with process changes: did adding that review step improve change failure rate?
- Compare across teams (carefully) — patterns reveal systemic issues vs team-specific ones
