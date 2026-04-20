# Best Practices

Proven Azure security practices. These are the controls that matter most, prioritized by impact.

## Zero Trust Foundation

Zero Trust is the security model, not an optional add-on. Every control below flows from these principles:

- **Verify explicitly** — authenticate and authorize based on all available signals: identity, location, device health, service, data classification, and anomalies. Never trust implicitly based on network location.
- **Use least privilege** — JIT/JEA access, risk-based adaptive policies, data protection. Grant the minimum permissions required for the minimum duration needed.
- **Assume breach** — minimize blast radius through segmentation. Verify end-to-end encryption. Use analytics for visibility and threat detection. Design every control assuming the adjacent control has already failed.

## Managed Identity Everywhere

- **Eliminate secrets from code** — every connection string, password, and API key in code is a liability. Replace with managed identity for all Azure service-to-service communication.
- **User-assigned for shared scenarios** — when multiple resources need identical permissions (e.g., multiple App Services accessing the same Storage account), use user-assigned managed identity to simplify RBAC management.
- **Workload identity federation for external** — GitHub Actions, AKS pods, and external CI/CD systems should use workload identity federation instead of service principal secrets.
- **Audit remaining secrets** — after migrating to managed identity, audit for any remaining connection strings or secrets. Target: zero secrets in application code.

## Private Endpoints for All Data Plane Access

- **Default to private** — every PaaS service that supports private endpoints should use them: Storage, SQL, Cosmos DB, Key Vault, Event Hubs, Service Bus, ACR. Public access should be disabled unless there's a documented business requirement.
- **DNS architecture** — centralize Private DNS Zones in a hub subscription with VNet links to all spokes. Use Azure Policy (DINE) to auto-create private endpoints and DNS records.
- **Verify with testing** — after configuring private endpoints, test resolution from within the VNet. Confirm `nslookup` returns private IP, not public.

## Defender for Cloud

- **Enable on all subscriptions** — Defender for Cloud CSPM (free tier) for all subscriptions. Enable Defender plans (Servers, Storage, SQL, Key Vault, Containers, App Service) for workload-specific protection.
- **Review recommendations weekly** — assign ownership for Secure Score improvement. Triage new recommendations within one business day. Remediate critical and high severity within defined SLAs.
- **Attack path analysis** — use Defender for Cloud's attack path analysis to identify multi-step attack vectors that combine individual weaknesses into exploitable chains. Prioritize these over isolated findings.
- **Continuous export to Sentinel** — stream Defender alerts and recommendations to Sentinel for correlation with other security signals and automated response.

## Key Vault for All Secrets

- **One vault per application per environment** — separates blast radius. Dev vault compromise doesn't expose production secrets. Simplifies RBAC and audit.
- **RBAC over access policies** — use Azure RBAC for Key Vault data plane. Access policies are legacy and don't support PIM or conditional access.
- **Enable purge protection** — prevents permanent deletion of secrets even by administrators during the retention period. Non-negotiable for production workloads.
- **Auto-rotation** — configure Key Vault auto-rotation for keys and certificates. Use Event Grid notifications for secrets that require application-side rotation logic.

## Conditional Access

- **Require MFA for all users** — start here. No exceptions except cloud-only break-glass accounts. This single control blocks the vast majority of identity attacks.
- **Block legacy authentication** — POP, IMAP, SMTP AUTH, and ActiveSync basic auth don't support MFA. Block them. Period.
- **Require compliant or Hybrid joined devices** — for sensitive applications, ensure devices meet organizational security standards via Intune compliance policies.
- **Named locations** — define trusted network locations. Require additional verification or block access from unexpected geographies.
- **Session controls** — enforce sign-in frequency (e.g., every 8 hours for sensitive apps), disable persistent browser sessions on unmanaged devices.

## RBAC Least Privilege

- **Built-in roles first** — custom roles only when no built-in role fits. Built-in roles are maintained by Microsoft and follow security best practices.
- **Assign at narrowest scope** — resource or resource group, not subscription. Subscription-level assignments are a security finding unless justified (e.g., platform team).
- **PIM for all privileged roles** — no permanent Owner, Contributor, or User Access Administrator. Require activation with justification, approval, and time-limited duration.
- **Regular access reviews** — quarterly at minimum. Use Entra ID access reviews for both Entra roles and Azure RBAC. Auto-remove denied access.

## Network Segmentation

- **Hub-spoke topology** — centralize security controls (Azure Firewall, Bastion, DNS) in hub. Workloads in isolated spoke VNets with NSGs on every subnet.
- **Azure Firewall for east-west and egress** — inspect traffic between spokes and outbound to internet. Use threat intelligence-based filtering and FQDN rules.
- **NSG flow logs** — enable on all NSGs and send to Log Analytics. Use Traffic Analytics for visibility into allowed and denied flows.
- **No direct internet access from workloads** — route all egress through Azure Firewall or NAT Gateway. No public IPs on VMs or application resources unless required (and document why).

## Encryption

- **At rest: CMK for sensitive workloads** — platform-managed keys (PMK) are the default and sufficient for many workloads. Customer-managed keys (CMK) via Key Vault for regulated data (PCI, HIPAA, financial).
- **In transit: TLS 1.2+ everywhere** — enforce minimum TLS version on all PaaS services. Disable TLS 1.0 and 1.1. Use Azure Front Door or Application Gateway for TLS termination with managed certificates.
- **Double encryption where available** — infrastructure encryption (encryption at the physical layer) plus service encryption for defense in depth on highly sensitive data.
- **Verify encryption status** — don't assume encryption is on. Use Azure Policy to audit and enforce encryption settings. Check Defender for Cloud recommendations.

## Logging and Monitoring

- **Diagnostic settings on everything** — enable diagnostic logs and metrics for all resources. Send to a centralized Log Analytics workspace. Minimum: Key Vault, Storage, SQL, NSGs, Entra ID sign-in and audit logs.
- **Centralized SIEM** — Microsoft Sentinel as SIEM/SOAR. Connect all data sources: Entra ID, Defender for Cloud, Azure Activity, Office 365, third-party. Enable built-in analytics rules.
- **Activity Log forwarding** — Activity Log auto-deletes after 90 days. Forward to Log Analytics for long-term retention and Sentinel for correlation.
- **Alert on security-critical events** — failed authentications, role assignment changes, Key Vault access from unexpected IPs, policy compliance changes. Automate response with Logic Apps or Sentinel playbooks.
- **Retention policy** — align log retention with regulatory requirements. Minimum 1 year for security logs, 7 years for compliance-sensitive industries.

## Incident Response Readiness

- **Documented runbooks** — pre-written response procedures for common scenarios: compromised credential, data exfiltration, ransomware, DDoS. Don't write procedures during an incident.
- **Break-glass accounts tested quarterly** — cloud-only accounts excluded from Conditional Access (except monitoring). Stored in a physical safe. Test the sign-in process regularly.
- **Automated containment playbooks** — Sentinel playbooks that automatically disable compromised accounts, isolate VMs, or revoke access tokens. Reduce response time from hours to seconds.
- **Regular tabletop exercises** — simulate security incidents with the full response team. Identify gaps in procedures, tooling, and communication before a real incident reveals them.

## Supply Chain Security

- **Scan container images in CI/CD** — integrate vulnerability scanning before images reach ACR. Block deployment of images with critical CVEs.
- **Dependency scanning** — enable GitHub Advanced Security or equivalent for all repositories. Alert on known vulnerabilities in third-party packages.
- **Signed and verified deployments** — use deployment stamps from trusted pipelines only. No manual deployments to production. Service connections scoped to specific resource groups.
