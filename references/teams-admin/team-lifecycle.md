# Teams Admin — Team Lifecycle Management

## Team Lifecycle Management

### Creation & Governance
- **Team creation policies**: Control who can create teams (Entra ID group-based restriction)
- **Naming conventions**: Prefix/suffix policies via Entra ID group naming policy (e.g., "Team-{Department}-{GroupName}")
- **Templates**: Pre-defined team structures with channels, apps, tabs, and settings
- **Sensitivity labels**: Apply at creation to set privacy, guest access, sharing controls

### Expiration & Archival
- **Expiration policy**: Auto-expire inactive teams (30, 60, 90, 180, 365 days, or custom)
  - Owners notified before expiration
  - Expired teams soft-deleted → 30-day recovery window
  - Set via Entra ID → Group expiration policy
- **Archival**: Read-only state — content preserved, no new messages/files
  - Archive via Teams Admin Center or PowerShell (`Set-TeamArchivedState`)
  - Archived teams still consume licenses and storage
  - Can be unarchived

### Classification & Labeling
- **Sensitivity labels** control: privacy, guest access, external sharing, unmanaged device access
- **Classification** (legacy): text-only labels, no enforcement
- Best practice: migrate from classification to sensitivity labels for enforcement
