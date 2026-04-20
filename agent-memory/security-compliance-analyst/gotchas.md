# Gotchas

Azure security behaviors that are technically correct but surprising. These cause production incidents.

## Managed Identity Lifecycle

- **System-assigned vs user-assigned lifecycle difference** — system-assigned identity is deleted when the resource is deleted. If you recreate the resource (e.g., redeployment), the identity changes and all RBAC assignments are lost. User-assigned identity survives resource deletion.
- **Slot-specific system-assigned identity** — App Service deployment slots each have their own system-assigned managed identity. The staging slot identity is different from production. RBAC assignments don't follow slot swaps.
- **Managed identity token caching** — tokens are cached and may take up to 24 hours to reflect RBAC changes. Restarting the resource forces token refresh. This causes "access denied" after granting permissions.

## Key Vault Soft Delete

- **Soft delete is now mandatory** — all Key Vaults created after February 2025 have soft delete enabled and cannot disable it. Deleted vaults remain for 7-90 days (configurable) and count against naming uniqueness.
- **Can't reuse Key Vault name during retention** — deleting a Key Vault and trying to create one with the same name in the same region fails until the soft-deleted vault is purged or the retention period expires.
- **Purge protection blocks permanent deletion** — when purge protection is enabled (recommended for production), even administrators cannot permanently delete secrets, keys, or certificates during the retention period. This is intentional but surprises teams during cleanup.

## Private Endpoints vs Management Plane

- **Private endpoint secures data plane only** — adding a private endpoint to a Storage account, SQL Database, or Key Vault secures data access. The management plane (ARM operations) still goes through public Azure endpoints.
- **DNS resolution is the real complexity** — private endpoints require correct DNS configuration. If DNS doesn't resolve to the private IP, traffic routes publicly even with private endpoint configured. Test with `nslookup` from within the VNet.
- **Private endpoint per service per VNet** — each PaaS service that needs private access from a VNet requires its own private endpoint and DNS zone integration. This scales to hundreds of endpoints in enterprise environments.

## Conditional Access Evaluation

- **Policies are AND'd not OR'd** — if multiple Conditional Access policies apply, all applicable policies must be satisfied. A policy requiring MFA AND a policy requiring compliant device means the user needs both. There is no "any one policy passes" logic.
- **Exclude before include** — exclusions are evaluated before inclusions. An excluded user or group is excluded regardless of any include rules. This means a misconfigured exclusion can bypass all security policies.
- **Report-only mode doesn't block** — report-only policies log what would have happened but don't enforce. Teams forget to switch from report-only to on, leaving gaps for months.
- **Continuous access evaluation (CAE)** — near-real-time token revocation for supported services. Without CAE, revoking a session can take up to 1 hour (token lifetime). Not all services support CAE yet.

## Azure Policy Deny vs Remediation

- **Deny blocks creation but doesn't fix existing resources** — a deny policy prevents new non-compliant resources but does nothing about resources deployed before the policy existed. Existing violations require remediation tasks.
- **DINE (DeployIfNotExists) requires managed identity** — remediation tasks need a managed identity with permissions to modify resources. If the identity lacks permissions, remediation silently fails.
- **Policy evaluation delay** — new or modified policies can take up to 30 minutes for initial evaluation. Compliance dashboard may show stale data during this window.

## Defender for Cloud Secure Score

- **Secure Score can be misleading** — exemptions and not-applicable assessments reduce the total weight denominator, artificially inflating the score. A 90% score with heavy exemptions is less secure than a 70% score with no exemptions.
- **Score is subscription-scoped** — each subscription has its own Secure Score. A high score on one subscription doesn't reflect the posture of others. Management group-level aggregation helps but isn't the default view.
- **Recommendations refresh is not instant** — after remediation, it can take 4-24 hours for the Secure Score to update. Teams remediate, see no change, and assume it didn't work.

## NSG vs Azure Firewall

- **NSG is L3-L4 only** — Network Security Groups filter by IP, port, and protocol. They cannot inspect application-layer traffic, URLs, or TLS content. For HTTP/HTTPS inspection, you need Azure Firewall Premium or a WAF.
- **NSG on subnet vs NIC** — NSGs can be applied at both subnet and NIC levels. Both are evaluated (most restrictive wins). NIC-level NSGs are often forgotten and cause unexpected blocks.
- **Default outbound access deprecation** — Azure is deprecating default outbound internet access. Resources without explicit outbound configuration (NAT Gateway, Azure Firewall, or Load Balancer with outbound rules) will lose internet access.

## RBAC Custom Role Deletion

- **Custom roles can't be deleted while assigned** — you must remove all role assignments referencing the custom role before deletion. With assignments spread across subscriptions, finding them all requires scripting (`Get-AzRoleAssignment`).
- **Custom role propagation delay** — changes to custom role definitions can take up to 15 minutes to propagate. During this window, the old permissions may still apply.

## Entra ID Guest User Enumeration

- **Guest users can enumerate the directory by default** — external guests can list users, groups, and applications in your Entra ID tenant. Restrict guest access in External Collaboration Settings to "most restrictive" to prevent this.
- **Guest user permissions are broader than expected** — by default, guest users have read access to user profiles, group memberships, and applications. Review and restrict via Entra ID > User Settings > External collaboration settings.

## Storage Account Key and SAS Token Interaction

- **Rotating one key doesn't invalidate active SAS tokens** — SAS tokens signed with key1 remain valid after key1 is rotated because the token was already generated. Only rotating the key used to sign the SAS invalidates it, and only for new validation attempts after the rotation propagates.
- **Account SAS vs service SAS vs user delegation SAS** — user delegation SAS (signed with Entra ID credentials) is the most secure. Account and service SAS are signed with storage keys and can't be individually revoked.

## Certificate-Based Authentication Failures

- **Certificate expiry causes silent auth failures** — certificate-based authentication does not produce user-friendly error messages when the certificate expires. Services simply fail to authenticate with generic errors. Monitor certificate expiry with Key Vault alerts or Entra ID notifications.
- **Certificate chain trust issues** — intermediate certificates must be included in the chain. Missing intermediates cause validation failures that appear as network errors, not authentication errors.
- **Clock skew sensitivity** — certificate validation is sensitive to time differences between client and server. VMs with drifted clocks fail certificate authentication silently.

## Resource Provider Registration

- **Features fail silently without provider registration** — enabling Defender plans, private endpoints, or diagnostic settings may fail if the required resource provider isn't registered on the subscription. Always check `Microsoft.Security`, `Microsoft.Network`, and `Microsoft.Insights` registration.
- **Resource provider registration is subscription-scoped** — registering a provider on one subscription doesn't register it on others. New subscriptions in a management group need explicit provider registration or Azure Policy to auto-register.

## Azure Firewall and UDR Interactions

- **Asymmetric routing breaks Azure Firewall** — if inbound traffic arrives via Azure Firewall but the return path bypasses it (no UDR on the subnet), the firewall drops the return traffic as it doesn't match a known session. UDRs must ensure symmetric routing.
- **Azure Firewall SNAT behavior** — Azure Firewall SNATs outbound traffic by default. Backend services see the Firewall's private IP, not the originating VM's IP. This affects IP-based access controls on downstream services.
- **Forced tunneling requires management subnet** — routing all traffic through Azure Firewall (0.0.0.0/0 UDR) requires a dedicated AzureFirewallManagementSubnet with its own public IP. Without it, the firewall loses management connectivity.

## Entra ID Token Lifetime

- **Access tokens are valid for 60-90 minutes by default** — revoking a user's access doesn't take effect until the current token expires. For near-real-time revocation, enable Continuous Access Evaluation (CAE) on supported services.
- **Refresh tokens survive password changes** — changing a user's password doesn't immediately revoke refresh tokens for all clients. Use `Revoke-AzureADUserAllRefreshToken` or the Entra admin portal to force re-authentication.
