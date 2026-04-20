# Anti-Patterns

Common Azure security anti-patterns that create systemic risk. Every one of these is a real finding from production environments.

## Security as Afterthought

- **Bolting security on after architecture is finalized** — security controls are exponentially harder and more expensive to retrofit. Network segmentation, private endpoints, and encryption strategies must be designed in, not patched on.
- **"We'll add security in the next sprint"** — this sprint never comes. Security debt compounds faster than technical debt because each new component inherits the insecure patterns of existing ones.
- **No threat model before deployment** — deploying without STRIDE analysis means you don't know what you're defending against. Every architecture decision is a security decision.

## Over-Permissioned Service Principals

- **Owner/Contributor at subscription scope** — a compromised service principal with Owner on a subscription can create new admins, delete resources, exfiltrate all data. Scope to resource group and use the minimal built-in role.
- **"We gave it Contributor because Reader didn't work"** — the answer is never to escalate to the broadest role. Find the specific action that failed and grant the narrowest role that includes it.
- **No PIM for privileged service accounts** — service principals with permanent privileged access are high-value targets. Use workload identity federation or certificate-based auth with short expiry.

## Shared Credentials

- **Multiple services using the same secret or key** — when one is compromised, all are compromised. No way to revoke access for one consumer without breaking the others.
- **Shared storage account keys across teams** — every team that has the key has full control of the entire storage account. Use RBAC with managed identity instead.
- **Single Key Vault shared across all environments** — dev, staging, and production secrets in one vault means a dev compromise exposes production. Separate vaults per environment.

## No Network Segmentation

- **Everything publicly accessible by default** — PaaS services with public endpoints exposed to the internet when only internal access is needed. Default should be private, not public.
- **Flat network topology** — all resources in one VNet with no NSGs means lateral movement is trivial after initial compromise. Use hub-spoke with NSGs on every subnet.
- **No private endpoints for data services** — Storage, SQL, Key Vault, Cosmos DB all support private endpoints. Public data plane access is an unnecessary attack surface.

## Ignoring Defender for Cloud

- **Dismissing recommendations without investigation** — Defender for Cloud recommendations represent real attack vectors. Dismissing them to improve Secure Score without remediation is self-deception.
- **Not enabling Defender plans** — the free CSPM tier misses workload-specific threats. Enable Defender for Servers, Storage, SQL, Key Vault, App Service, and Containers.
- **No one monitors the alerts** — Defender for Cloud generating alerts that go to an unmonitored inbox is the same as having no alerts. Route to Sentinel or a monitored SIEM.

## Secrets in Code and Config

- **Connection strings in appsettings.json committed to source control** — secrets in code repos are permanent. Even after rotation, the old secret is in git history forever.
- **Secrets in environment variables without Key Vault reference** — environment variables are visible in App Service configuration, Function App settings, and container orchestrator configs. Use Key Vault references.
- **Hardcoded SAS tokens** — SAS tokens in code cannot be revoked (only the underlying key rotation invalidates them). Use managed identity or short-lived SAS generated at runtime.

## No Encryption Strategy

- **Relying on defaults without verification** — Azure encrypts at rest with platform-managed keys by default, but this isn't sufficient for regulated workloads. Verify encryption is actually enabled and use CMK where required.
- **No TLS enforcement** — allowing HTTP or TLS 1.0/1.1 when TLS 1.2+ should be the minimum. Enforce minimum TLS version on all services.
- **No encryption in transit between internal services** — "it's inside our VNet" is not a defense. Assume breach means internal traffic must also be encrypted.

## Disabling MFA

- **MFA bypassed for "convenience"** — MFA is the single most effective control against identity compromise. Bypassing it for any user, especially admins, is accepting the highest-probability attack vector.
- **Service accounts without MFA or Conditional Access** — service accounts are high-value targets. Use managed identity (no MFA needed) or workload identity federation. If passwords are unavoidable, Conditional Access with location restrictions.
- **Break-glass accounts without monitoring** — break-glass accounts that bypass MFA must have sign-in alerts routed to the security team immediately.

## Not Rotating Secrets

- **Secrets and certificates with no rotation schedule** — a secret that has never been rotated has been valid since creation, maximizing the window of opportunity if compromised.
- **No automated rotation** — manual rotation is error-prone and gets deferred. Use Key Vault auto-rotation for certificates and keys. Automate secret rotation with Azure Functions or Event Grid triggers.
- **Rotating keys but not SAS tokens** — rotating a storage account key does not invalidate SAS tokens generated from the other key. Both keys must rotate, and SAS tokens must be regenerated.

## Compliance Checkbox Mentality

- **Meeting minimum requirements without understanding threats** — passing a PCI-DSS audit does not mean you're secure. Compliance is a floor, not a ceiling. Attackers don't care about your audit report.
- **Copying policy definitions without understanding effects** — deploying Azure Policy initiatives without understanding deny vs audit vs DINE effects leads to either blocking legitimate deployments or providing false assurance.
- **Annual security reviews only** — threat landscapes change weekly. Annual reviews find issues 6-12 months too late.

## Single Admin Without Break-Glass

- **One admin account for the entire tenant** — if that account is compromised or locked out, total loss of control. If the admin leaves the organization, recovery requires Microsoft support.
- **No break-glass procedure** — break-glass accounts must exist, be cloud-only (no federation dependency), excluded from Conditional Access except monitoring, and tested quarterly.
- **Break-glass credentials stored in the same system they protect** — if your Key Vault is compromised and break-glass creds are in Key Vault, you're locked out. Store offline in a physical safe.

## Not Logging Security Events

- **No diagnostic settings on critical resources** — Key Vault, Storage, SQL, and Entra ID sign-in logs must be captured. Without logs, incident response is guesswork.
- **Logs retained for 30 days only** — regulatory requirements often mandate 1-7 years. Attackers frequently dwell for months before detection. Short retention means evidence is gone.
- **Activity Log not forwarded to SIEM** — Activity Log captures control plane operations but auto-expires after 90 days. Forward to Log Analytics and Sentinel for correlation and long-term retention.
- **No alerting on security-critical events** — logging without alerting is an archive, not a security control. Alert on failed admin sign-ins, role assignment changes, policy compliance drift, and Key Vault access anomalies.

## Neglecting Supply Chain Security

- **No container image scanning** — deploying container images without vulnerability scanning allows known CVEs into production. Enable Defender for Containers and scan in CI/CD pipelines before push to ACR.
- **No dependency scanning in CI/CD** — third-party packages are a primary attack vector. Use GitHub Advanced Security or similar tools to scan for vulnerabilities in NuGet, npm, PyPI, and Maven dependencies.
- **Pulling images from public registries in production** — Docker Hub and other public registries are untrusted. Import approved images to your Azure Container Registry and pull only from ACR in production.

## Ignoring Subscription Hygiene

- **Single subscription for all workloads** — mixing production, dev, and sandbox in one subscription means a dev misconfiguration can impact production. Use separate subscriptions per environment with management group policies.
- **No resource locks on critical resources** — accidental deletion of a production database or Key Vault should be impossible. Apply CanNotDelete locks on critical resources.
- **No tagging strategy** — without consistent tags, you can't identify resource ownership, cost center, or data classification. Untagged resources are unmanaged resources.
