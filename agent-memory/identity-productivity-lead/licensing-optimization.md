## Licensing Optimization

### E3 vs E5 Feature Comparison

| Capability Area | E3 Includes | E5 Adds |
|----------------|-------------|---------|
| **Identity** | Entra ID P1 (Conditional Access, MFA, self-service password reset) | Entra ID P2 (PIM, Identity Protection, access reviews) |
| **Threat Protection** | Defender for Office 365 P1, basic Defender for Endpoint | Defender for Office 365 P2, full Defender for Endpoint, Defender for Identity, Defender for Cloud Apps |
| **Information Protection** | Basic DLP, manual sensitivity labels, basic retention | Auto-labeling, advanced DLP, insider risk management, advanced eDiscovery |
| **Compliance** | Basic audit, content search, basic retention policies | Advanced audit (long-term retention), communication compliance, information barriers |
| **Analytics** | Basic M365 usage reports | Power BI Pro, Viva Insights advanced, MyAnalytics |
| **Voice** | Teams meetings and chat | Phone System, Audio Conferencing |

### Add-On Strategy

**E3 + targeted add-ons vs full E5** — the core licensing decision:

- **Break-even calculation**: If an organization needs 3+ E5-only features, full E5 is typically cheaper than E3 plus individual add-ons
- **E5 Security add-on**: E3 + Defender suite + Entra P2 without compliance features — for security-focused organizations
- **E5 Compliance add-on**: E3 + advanced Purview + insider risk without security features — for compliance-focused organizations
- **Partial E5 deployment**: Not every user needs E5 — assign E5 to admins, executives, high-risk users and E3 to standard users

### Group-Based License Assignment

Assign licenses through Entra ID groups rather than per-user:

1. **Create license groups**: `LIC-M365-E3`, `LIC-M365-E5`, `LIC-E5-Security-AddOn`
2. **Map to organizational roles**: Standard users → E3 group, IT admins → E5 group, finance/legal → E5 Compliance add-on
3. **Automate with dynamic groups**: Department-based or attribute-based membership rules
4. **Handle conflicts**: Entra ID reports license assignment errors — monitor and resolve overlapping assignments
5. **Disable unused service plans**: Turn off individual services within a license (e.g., disable Sway, Kaizala) to simplify the user experience

### License Audit Practices

**Identifying waste — run quarterly:**

- **Unused E5 features**: Users with E5 who never use P2, Defender, or advanced Purview features → downgrade candidates
- **Standalone add-on overlap**: Users with E3 + multiple add-ons that cost more than E5 → upgrade candidates
- **Service account licenses**: Accounts consuming user licenses that could use app registrations or managed identities instead
- **Shared mailbox licensing**: Shared mailboxes under 50GB don't require licenses — remove unnecessary assignments
- **Departed user licenses**: Licenses still assigned to disabled accounts → reclaim within offboarding workflow
- **Inactive users**: Accounts with no sign-in for 90+ days — investigate and reclaim

### Cost Per User Optimization

| Action | Typical Savings |
|--------|----------------|
| Downgrade unused E5 to E3 | ~$20/user/month |
| Remove licenses from inactive accounts | Full license cost recovered |
| Replace standalone add-ons with bundled E5 | Variable — calculate per scenario |
| Use shared mailboxes instead of licensed mailboxes | ~$12-36/user/month |
| Segment E5 to high-risk users only | Significant when E5 population drops below 40% |

### Reporting and Governance

- **M365 admin center → Usage reports**: Per-app activation and usage data
- **Entra ID → Sign-in logs**: Last sign-in date per user to identify inactive accounts
- **PowerShell/Graph API**: Automate license inventory and usage reporting
- **License dashboard**: Build a recurring report showing assigned vs consumed vs available licenses
- **Budget alignment**: Map license costs to departments using cost center tags on license groups
