# Well-Architected Framework — Cost Optimization & Operational Excellence

## Cost Optimization Checklist for Apps

- [ ] Right-size compute (actual usage vs provisioned)
- [ ] Auto-scale configured with min/max bounds
- [ ] Dev/test environments on cheaper SKUs or shut down on schedule
- [ ] Reserved instances or savings plans for steady-state production
- [ ] Spot instances for fault-tolerant batch workloads
- [ ] Storage lifecycle policies to tier cold data
- [ ] Egress costs estimated (cross-region, cross-zone, internet)
- [ ] Tagging strategy enforced for cost allocation

## Operational Excellence Checklist

- [ ] IaC for all infrastructure (no portal deployments in production)
- [ ] CI/CD pipelines with automated testing gates
- [ ] Diagnostic settings on every resource shipping to Log Analytics
- [ ] Alerts configured for SLO-breaching conditions
- [ ] Runbooks for common failure scenarios
- [ ] Deployment frequency and change failure rate tracked
- [ ] Feature flags for progressive rollout
