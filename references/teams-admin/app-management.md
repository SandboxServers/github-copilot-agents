# Teams Admin — App Management

## Teams App Types

### Categories
| Type | Description | Examples |
|------|-------------|---------|
| **Microsoft apps** | Built by Microsoft, included with Teams | Planner, Forms, SharePoint, OneNote, Approvals |
| **Third-party apps** | Published by ISVs in the Teams App Store | ServiceNow, Salesforce, Jira, Adobe Sign |
| **Custom apps (LOB)** | Built by your organization for internal use | Custom bots, tabs, connectors, message extensions |

## App Permission Policies

### Purpose
- Control which apps users can install and use in Teams
- Separate controls for each app type: Microsoft, third-party, custom

### Configuration
- Teams Admin Center → Teams apps → Permission policies
- Actions per app type: Allow all apps, Block all apps, Allow specific apps, Block specific apps
- Assigned to users via direct assignment or group assignment

### Behavior
- If an app is blocked by permission policy, users cannot install or use it
- Existing installations are disabled but not removed when policy changes
- Global (org-wide default) policy applies to users without explicit assignment

## App Setup Policies

### Purpose
- Control which apps are pinned (pre-installed) in users' Teams app bar
- Define the order of pinned apps in the left navigation

### Configuration
- Teams Admin Center → Teams apps → Setup policies
- Pin apps: select apps to appear in the app bar for assigned users
- Upload custom apps: allow or block users from uploading custom/LOB apps
- Install apps: pre-install specific apps for users without requiring user action

## Org-Wide App Settings

### Global Controls
- Teams Admin Center → Teams apps → Manage apps → Org-wide app settings
- Third-party apps: allow or block all third-party apps across the org
- Custom apps: allow or block all custom app uploads across the org
- These are master switches — if disabled, per-user policies cannot override

## Custom App Submission and Approval

### Submission Flow
1. Developer uploads app package (.zip) to Teams Admin Center or via Teams client
2. App appears in "Manage apps" with status "Submitted" (pending review)
3. Admin reviews: app manifest, permissions requested, publisher information
4. Admin approves (publishes to org store) or rejects

### Custom App Store
- Approved custom apps appear in "Built for your org" section of Teams App Store
- Users install from org store subject to their app permission policy
- Version updates: developer submits new package, admin approves update

## App Centric Management

### Per-App Settings
- Teams Admin Center → Teams apps → Manage apps → select individual app
- Control per app: who can install (everyone, specific users/groups, no one)
- View app details: publisher, permissions, description, certification status
- Override permission policies for individual apps without changing policy

### App Certification
- Microsoft 365 Certification: app has passed security and compliance review
- Publisher Attestation: publisher self-attests to security practices
- Neither certification → uncertified (may require extra review before allowing)

## Teams App Store Governance

### Best Practices
- Start restrictive: block all third-party and custom apps, allow selectively
- Review apps before allowing: check permissions, data access, publisher reputation
- Use app permission policies to scope access by department or role
- Monitor app usage via Teams Admin Center → Analytics & reports → App usage
- Regularly audit installed apps and remove unused or risky apps