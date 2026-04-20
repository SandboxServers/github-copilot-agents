## DORA Metrics — Measuring Software Delivery Performance

The DevOps Research and Assessment (DORA) program identified four key metrics that reliably predict software delivery performance and organizational outcomes. These are the industry standard for understanding how well your engineering organization delivers software.

### The Four Key Metrics

**Deployment Frequency** measures how often your organization deploys code to production. Higher frequency indicates smaller batch sizes, better automation, and greater confidence in the release process. Teams deploying on-demand ship smaller, safer changes.

**Lead Time for Changes** measures the elapsed time from code commit to that commit running in production. This captures the total pipeline efficiency — build time, test time, approval wait time, and deployment time. Short lead times mean fast feedback loops and rapid value delivery.

**Change Failure Rate** measures the percentage of deployments that cause a failure in production requiring remediation — a rollback, hotfix, or patch. This is the quality counterpart to speed. Deploying fast but breaking things constantly is not high performance.

**Mean Time to Recover (MTTR)** measures how quickly an organization can restore service after a production incident. Fast recovery requires good monitoring, clear runbooks, automated rollback capability, and practiced incident response.

### Performance Benchmarks

| Metric | Elite | High | Medium | Low |
|--------|-------|------|--------|-----|
| Deployment Frequency | On-demand, multiple per day | Weekly to monthly | Monthly to semi-annually | Less than once per 6 months |
| Lead Time for Changes | Less than one hour | One day to one week | One month to six months | More than six months |
| Change Failure Rate | 0–15% | 16–30% | 16–30% | 46–60% |
| Mean Time to Recover | Less than one hour | Less than one day | One day to one week | More than six months |

Elite performers excel at all four simultaneously. Speed and stability are not trade-offs — they reinforce each other through disciplined automation, testing, and operational practices.

### Measuring in Azure DevOps

- **Deployment Frequency**: Query pipeline run history filtered by production stage completions using the Analytics service or REST API
- **Lead Time**: Use the lead time widget in Azure Boards combined with pipeline timestamps — commit time from the build artifact to production deployment completion
- **Change Failure Rate**: Tag failed deployments in release pipelines and compute the ratio against total production deployments over a rolling window
- **MTTR**: Track incident work items linked to deployments — measure time from incident creation to resolution

### Measuring in GitHub

- **Deployment Frequency**: Query the Deployments API filtered by production environment, aggregate by time period
- **Lead Time**: Compare commit timestamp from the deployment payload against deployment completion time in GitHub Actions workflow runs
- **Change Failure Rate**: Use deployment statuses (failure vs success) from the Deployments API or track rollback workflows
- **MTTR**: Correlate GitHub Issues labeled as incidents with deployment events — measure open-to-close duration

### Using Metrics Effectively

Metrics exist to drive improvement, not to punish teams. Share DORA data transparently so teams can identify their own bottlenecks. A team with a high change failure rate needs better testing, not a performance review. A team with long lead times needs pipeline optimization, not pressure to skip steps.

Track trends over quarters, not days. Celebrate improvements rather than targeting absolute numbers. Use metrics to justify investments — if lead time is six months, that is the argument for pipeline automation funding.

### Beyond the Four Keys

Additional signals that complement DORA:
- **Deployment pain**: Survey-based — do engineers dread deployments or consider them routine?
- **Developer satisfaction (DevEx)**: Measures friction, toil, and cognitive load in the development process
- **Cost per deployment**: Infrastructure and tooling cost divided by deployment count — efficiency indicator
- **Unplanned work ratio**: Percentage of effort spent firefighting versus planned delivery
- **Mean time between failures (MTBF)**: Stability indicator that complements MTTR

### Common Pitfalls

- Measuring only deployment frequency while ignoring failure rate — speed without quality
- Comparing teams with fundamentally different workloads — a data pipeline team and a web frontend team have different deployment profiles
- Gaming metrics by deploying no-op changes or splitting deployments artificially
- Using DORA as a performance evaluation tool rather than a continuous improvement framework
- Measuring at the wrong granularity — organizational averages hide team-level problems
