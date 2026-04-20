## Power Apps

### Canvas Apps
- Pixel-level UI control with drag-and-drop WYSIWYG designer
- Connect to 400+ data sources (SharePoint, SQL, Dataverse, REST APIs, Excel)
- Power Fx formula language for logic, navigation, and data manipulation
- Best for task-specific apps with custom UI requirements
- Responsive design must be manually implemented with containers
- Ideal when integrating multiple non-Dataverse data sources
- Full creative control — every pixel, every layout decision is yours

### Model-Driven Apps
- Built on Dataverse — data model drives the UI automatically
- Form-driven: main forms, quick-create forms, quick-view forms
- Consistent, accessible UX with built-in navigation and sitemap
- Automatic relationship handling from Dataverse schema
- Best for data-heavy business apps (CRM-style CRUD operations)
- Rapid development once the data model exists
- Views, charts, and dashboards configured declaratively

### Canvas vs Model-Driven Decision

| Aspect | Canvas | Model-Driven |
|---|---|---|
| **UI control** | Pixel-perfect, fully custom | Component-driven, consistent |
| **Data sources** | Any connector (400+) | Dataverse only |
| **Design approach** | WYSIWYG + Power Fx | Driven by data model |
| **Responsive** | Manual effort | Automatic |
| **Relationship nav** | Must be coded | Built-in |
| **Best for** | Task-specific, mobile, custom UX | Forms-over-data, business process |

### Custom Pages
- Canvas app capabilities embedded inside model-driven apps
- Standard MDA navigation + custom canvas screens in one app
- Better integration and performance than legacy embedded canvas apps
- Limit to ~25 custom pages per app (performance degrades beyond)
- Use for screens needing custom layout within a model-driven shell

### Component Libraries
- Reusable UI components shared across multiple canvas apps
- Build once, update centrally — changes propagate to consuming apps
- Custom properties (input/output) for configurable behavior
- Store in solutions for proper ALM and environment promotion
- Use for headers, navigation menus, common form patterns, branded controls

### App Performance

**Delegation**
- Push query processing to the data source instead of loading all data to client
- Non-delegable functions pull max 500/2000 records then filter locally — silent data loss
- Always check delegation warnings (yellow triangles in formula bar)
- Key delegable functions: Filter, Sort, LookUp (varies by data source)
- Restructure formulas to be delegable — avoid `Search()` on large datasets

**Concurrent() Function**
- Execute multiple data calls simultaneously instead of sequentially
- Use in App.OnStart or screen OnVisible for parallel data loading
- Dramatically reduces app load time when fetching from multiple sources
- Syntax: `Concurrent(ClearCollect(col1, src1), ClearCollect(col2, src2))`

**Caching Strategies**
- Use collections to cache reference data that rarely changes
- Load once with `ClearCollect()`, work from local collection afterward
- Implement manual refresh button for user-controlled cache invalidation
- Balance freshness vs performance — don't cache transactional data

### Naming Conventions
- Screens: `scrHome`, `scrDetail`, `scrSettings`
- Buttons: `btnSubmit`, `btnCancel`, `btnNavigate`
- Variables: `varUserName`, `varSelectedRecord`
- Collections: `colEmployees`, `colLookupData`
- Name everything — never leave Button1, Label37, Gallery3
