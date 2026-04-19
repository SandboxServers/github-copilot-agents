## Organization Structure

### Four Divisions

| Division | Lead Agent | Specialists | Domain |
|----------|-----------|-------------|--------|
| **Platform Engineering** | @platform-engineering-lead | azure-apps-infra-architect, landing-zone-specialist, caf-evangelist, azure-terraform-author, bicep-code-author | Azure infrastructure foundations, landing zones, governance, IaC, CAF alignment |
| **DevOps** | @devops-lead | azure-pipelines-architect, github-actions-architect, ado-github-migration, azure-terraform-author | CI/CD, pipeline architecture, developer experience, golden paths, DORA metrics, platform migrations |
| **Azure Specialty** | @azure-specialty-lead | azure-ai-specialist, azure-data-engineer, azure-storage-engineer, azure-compute-engineer, azure-network-engineer, azure-monitoring-engineer, azure-integration-architect, azure-database-specialist | Deep Azure domain expertise, cross-domain solutions, technology selection, architecture design |
| **Identity & Productivity** | @identity-productivity-lead | entra-id-specialist, teams-admin, m365-engineer | Microsoft 365, Entra ID, zero trust, collaboration, licensing, adoption |

### Shared Services (Mandatory Gates)

| Service | Agent | Authority | Gate Type |
|---------|-------|-----------|-----------|
| **Security & Compliance** | @security-compliance-analyst | **VETO POWER** — nothing ships without security sign-off | Every phase gate. If security says no, the answer is no. |
| **Cost Optimization** | @cost-optimization-specialist | **BUDGET APPROVAL** — must approve new Azure spend before provisioning | Before any resource provisioning. If cost says the budget doesn't work, the design changes. |
| **Documentation** | @documentation-writer | **CLOSE GATE** — engagement cannot close without completed documentation | Before engagement closure. If docs aren't done, the engagement isn't done. |
| **Automation** | @powershell-automation-dev | Builds operational automation for every engagement | During implementation and handoff phases |
| **Testing & Validation** | @testing-validation-engineer | Validates infrastructure and application deployments | Before production deployment |
| **Retrospective** | @retrospective-agent | Post-engagement review and lessons learned | After engagement closure |
