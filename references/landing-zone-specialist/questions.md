## Questions This Specialist Always Asks

### Foundation
- Greenfield or brownfield? (If brownfield: how many subscriptions, what MG structure exists?)
- What's the network topology today? (On-prem, existing Azure, hybrid connectivity)
- What IaC tooling does the team use? (Terraform, Bicep, ARM, or ClickOps?)
- What's the deployment pipeline? (Azure DevOps, GitHub Actions, other?)
- Any compliance requirements? (FedRAMP, HIPAA, PCI, SOX, data residency)

### Network (the hard-to-change decisions)
- How many Azure regions? (Now and planned — affects hub topology)
- On-prem connectivity method? (ExpressRoute, VPN, both, none?)
- How many branch offices need Azure connectivity? (>30 → consider VWAN)
- IP address space available? (Show me the IPAM plan. If there isn't one, that's priority #1.)
- Do you need spoke-to-spoke direct communication? (Affects topology choice)

### Governance
- What policies are in place today? (If "none" → start with audit)
- How do teams get subscriptions today? (If "email request + 3 weeks" → subscription vending)
- What's the tagging strategy? (If "we don't have one" → define before deploying policies)
- Who handles exemptions? (There MUST be an exemption process before enforcing Deny policies)

### Identity
- How many Global Admins? (If >5 → reduce immediately)
- Break-glass accounts? (If "what's that?" → create them before anything else)
- PIM enabled? (If not → enable for all privileged roles)
- Service principals: how many, who owns them, when do credentials expire?
- Conditional access policies? (If "none" → baseline policies are day-one priority)
