# M365 Administration Gotchas

Non-obvious behaviors and subtle distinctions that frequently trip up Microsoft 365 administrators.

---

## License Assignment — Feature Dependency Chains

- Some features require specific license combinations that aren't obvious — e.g., Microsoft Defender for Endpoint Plan 2 requires Intune enrollment for full remediation capabilities
- Sensitivity labels with auto-labeling require Microsoft 365 E5 or E5 Compliance add-on — E3 only gets manual labeling
- Copilot for Microsoft 365 requires Microsoft 365 E3/E5 **plus** a separate Copilot add-on license — not included in any base plan
- Disabling a service plan within a license SKU (e.g., turning off Exchange Online in E5) can break dependent features like Teams voicemail or eDiscovery
- Group-based licensing in Entra ID can silently fail if insufficient licenses remain — check the Entra ID licensing blade for assignment errors regularly

## SharePoint Storage — Pooled, Not Per-Site

- SharePoint Online storage is **pooled per tenant**: 1 TB base + 10 GB per licensed user — not allocated per site
- A single site with large video or media libraries can exhaust the entire tenant pool, blocking all other sites from uploading
- Per-site storage quotas exist but are not set by default — without quotas, first-come-first-served consumption applies
- Additional storage can be purchased in 1 GB increments but is expensive ($0.20/GB/month) — archive or offload to Azure Blob before buying
- OneDrive storage (1 TB or 5 TB per user) is separate from SharePoint pool but counts against tenant totals for compliance features

## Exchange Online — Mail Flow Connector Complexity

- Inbound and outbound connectors interact with transport rules in non-obvious order: **inbound connector → transport rules → outbound connector**
- When using a third-party email security gateway, connector misconfiguration causes mail loops, NDRs, or SPF failures
- Enhanced filtering for connectors (skip listing) is required for Defender for Office 365 to see the true source IP — without it, all mail appears from the gateway IP
- Shared mailboxes don't require a license for basic use, but auto-mapping to more than ~15 users causes Outlook performance degradation
- Exchange Online Protection (EOP) anti-spam tuning: SCL (Spam Confidence Level) thresholds differ between standard and strict preset policies — overriding one resets the other

## Intune — Compliance Policy Evaluation Order

- Multiple compliance policies assigned to a user/device are **merged** — the most restrictive setting wins across all policies
- If a compliance setting is configured in one policy but "Not configured" in another, the configured value applies (Not configured ≠ off)
- Mark device non-compliant action timer: the default "Immediately" can lock users out during enrollment — set a grace period (24-72 hours) for new enrollments
- Compliance evaluation frequency is every 8 hours by default — users can trigger manual sync, but policy changes don't take effect instantly
- Platform-specific gotcha: Android Enterprise work profile compliance differs from fully managed — policies aren't interchangeable

## Purview — Sensitivity Label Propagation Delay

- Newly published sensitivity labels or label policy changes take **up to 24 hours** to appear in Office apps
- Auto-labeling policies in simulation mode show results within hours, but switching to enforcement has its own propagation delay
- Labels applied to SharePoint sites (container labels) take effect on the site settings immediately, but downloading labeled files respects the file-level label, not the container label
- Removing a label from a published policy doesn't remove it from already-labeled content — content retains the label until manually changed or a new policy is applied
- Sensitivity label priority (order number) determines which label wins in auto-labeling conflicts — higher priority number overrides lower

## Power Platform — DLP Policies Scope

- DLP policies apply **at the environment level**, not tenant-wide by default — you must explicitly include environments in each policy
- A tenant-level DLP policy exists but acts as a fallback for environments without a specific policy — it doesn't override environment-level policies
- Connector classification (Business, Non-Business, Blocked) in DLP is per-policy — the same connector can be Business in one environment and Blocked in another
- Custom connectors are unclassified by default — they bypass DLP unless explicitly added to a connector group
- HTTP connector and HTTP with Microsoft Entra ID are separate connectors — blocking one doesn't block the other

## Multi-Geo — Cost and Complexity

- Multi-Geo adds per-user cost for satellite locations — only users with data residency requirements should be assigned satellite geo
- Mailbox moves to satellite geos take up to 72 hours and require careful planning around calendar and mail flow
- SharePoint site geo moves are admin-initiated and block user access during the move window (hours to days depending on size)
- Search in Multi-Geo returns results from all geos but with latency for cross-geo queries — users may not see satellite content immediately
- eDiscovery and compliance features operate per-geo — multi-geo eDiscovery requires searching each geo separately

## Conditional Access + Intune — License Prerequisites

- Device compliance as a Conditional Access grant control requires **Entra ID P1 + Intune** — Entra ID P1 alone isn't sufficient
- Named locations using GPS-based detection require the Microsoft Authenticator app configured for location — IP-based named locations don't
- Conditional Access "Require approved client app" and "Require app protection policy" are different controls — approved app checks the app binary, app protection checks MAM policy enrollment
- Report-only mode is essential for testing but doesn't block anything — misconfiguring a CA policy in enforcement without testing first causes mass lockouts

## OneDrive — Known Folder Move (KFM) Gotchas

- KFM silently redirects Desktop, Documents, and Pictures to OneDrive — but files exceeding OneDrive's 250 GB sync limit or containing unsupported characters fail silently
- OneNote notebooks in redirected folders cause sync conflicts — OneNote uses its own sync engine and doesn't work well in KFM-redirected paths
- KFM configured via Intune policy (or GPO) requires the user to be signed into OneDrive — if OneDrive sync client isn't running, the policy queues but doesn't redirect
- Files with path lengths exceeding 400 characters are skipped — users don't receive obvious warnings
- Existing folder contents over 100,000 items can cause the initial KFM sync to take days

## Exchange Hybrid — Directory Sync Timing

- Directory sync (Entra ID Connect) runs every 30 minutes by default — mailbox provisioning changes are not instant
- Creating a remote mailbox on-premises and waiting for sync means the user has no mailbox for up to 30 minutes (or until forced sync)
- Mail routing (MX record) changes during hybrid cutover must account for DNS TTL — lowering TTL 48 hours before cutover prevents mail delivery delays
- Distribution group membership changes synced from on-premises take one sync cycle — urgent changes require `Start-ADSyncSyncCycle -PolicyType Delta`
- Hybrid Free/Busy lookups require OAuth or DAUTH configuration — misconfigurations show "no information available" without error messages

## SharePoint — Admin vs Owner Permissions

- **Site collection administrator** is a SharePoint-specific role with full control over a site collection — it bypasses all permission settings within that site
- **Site owner** is a SharePoint permission level — owners can manage lists, libraries, and site settings but cannot manage site collection-level features (search schema, site policies, recycle bin second stage)
- Adding someone as a site collection admin does NOT add them to the Owners SharePoint group — these are independent permission paths
- Site collection admins can access any content in the site regardless of item-level or library-level permissions — this is by design for admin recovery but surprises teams expecting access control isolation
- Global Administrators and SharePoint Administrators in Entra ID can make themselves site collection admin on any site — but they're not automatically listed as one
