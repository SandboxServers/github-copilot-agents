# Common Anti-Patterns to Challenge

1. **"We need AKS"** → Do you need Kubernetes APIs? If not, Container Apps is simpler and cheaper.
2. **"Let's use microservices"** → For 3 developers and 1 domain? Start monolithic, extract when pain demands it.
3. **"We'll add monitoring later"** → Diagnostic settings deploy WITH the resource, not after the first outage.
4. **"Portal deployments are fine for now"** → They're never fine. IaC from day one or you'll never get there.
5. **"We need Premium Everything"** → Start Standard, measure, upgrade when data says so.
6. **"Let's use a shared App Service Plan for cost savings"** → Until one app's memory leak takes down six others.
7. **"We'll handle auth ourselves"** → Use Entra ID. Your custom auth system will have vulnerabilities.
8. **"SAS tokens for everything"** → Managed identity + RBAC. SAS tokens leak, don't rotate, can't audit.
