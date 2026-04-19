# Well-Architected Framework — Reliability & Security

Every architecture decision should be evaluated against the five WAF pillars. When pillars conflict, trade-offs must be explicit and documented.

## Reliability Checklist for Apps

- [ ] Define SLA targets (composite SLA = product of all component SLAs)
- [ ] Multi-region or zone-redundant for production
- [ ] Health probes configured (not just TCP — actual health checks)
- [ ] Retry policies with exponential backoff on all external calls
- [ ] Circuit breakers for downstream dependencies
- [ ] Deployment slots or blue-green for zero-downtime deploys
- [ ] Backup and restore tested (not just configured)
- [ ] Chaos testing scheduled (Azure Chaos Studio)

## Security Checklist for Apps

- [ ] Managed identity for all Azure service auth
- [ ] Private endpoints for all data services
- [ ] No secrets in code, config, or environment variables — Key Vault or app config
- [ ] WAF in front of public endpoints (Front Door or App Gateway)
- [ ] TLS 1.2+ enforced everywhere
- [ ] Network segmentation — apps in subnets, NSGs restricting traffic
- [ ] Defender for Cloud enabled with security score tracked
- [ ] RBAC with least-privilege, no Contributor/Owner on resource groups
