## DORA Metrics — The Four Keys

The DevOps Research and Assessment (DORA) metrics are the industry standard for measuring software delivery performance:

| Metric | Definition | Elite | High | Medium | Low |
|--------|-----------|-------|------|--------|-----|
| **Deployment Frequency** | How often code deploys to production | On-demand (multiple per day) | Weekly to monthly | Monthly to semi-annually | Less than once per 6 months |
| **Lead Time for Changes** | Time from commit to production | Less than one hour | One day to one week | One month to six months | More than six months |
| **Change Failure Rate** | % of deployments causing failure | 0-15% | 16-30% | 16-30% | 46-60% |
| **Time to Restore Service (MTTR)** | Time to recover from production failure | Less than one hour | Less than one day | One day to one week | More than six months |

### Measuring DORA in Practice

- **Azure DevOps**: Use Analytics service with lead time and cycle time widgets, pipeline failure/duration reports, and cumulative flow diagrams
- **GitHub**: Use GitHub Actions workflow run history, deployment frequency from environments, and custom dashboards via the API
- **Cross-platform**: Aggregate metrics when teams use both platforms — the organizational view matters more than any single pipeline

### Beyond DORA

Additional metrics that matter:
- **Platform capability adoption**: % of teams using shared templates, number of repos on paved paths
- **Cost per deployment**: Infrastructure cost of CI/CD divided by deployment count
- **Developer satisfaction (DevEx)**: Survey-based, measures friction and toil
- **Mean time between failure (MTBF)**: Stability indicator
- **Unplanned work percentage**: Ratio of firefighting to planned delivery
