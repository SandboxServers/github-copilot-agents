## Power Apps

### Canvas Apps vs Model-Driven Apps

| Aspect | Canvas Apps | Model-Driven Apps |
|---|---|---|
| **Data platform** | Dataverse + 400+ connectors | Dataverse only |
| **UI control** | Full control (pixel-perfect) | Limited — component-focused customization |
| **Design approach** | WYSIWYG, Power Fx expressions | No-code, driven by data model + components |
| **Responsive** | Only if designed that way | Automatically responsive |
| **Speed to build** | Depends on design complexity | Rapid once data model exists |
| **Navigation** | Custom, designed per app | Built-in, consistent across all MDA |
| **Accessibility** | Must be designed in | Built-in |
| **Migration between environments** | Potentially complex (datasource updates) | Simple via solutions |
| **Consistency** | Low — every maker builds differently | High — uniform UX across all apps |
| **Relationships** | Must be coded with Power Fx | Automatic if Dataverse relationships exist |

### When to Use Which

**Canvas apps** when:
- You need pixel-perfect custom layouts
- You're integrating data from multiple non-Dataverse sources
- You need a very specific mobile experience
- The UX requirements are unique to this scenario

**Model-driven apps** when:
- Data lives in Dataverse (or should)
- You need forms-over-data CRUD applications
- Speed of delivery matters more than pixel-perfect design
- You want consistent, accessible UX out of the box
- You need built-in relationship navigation

**Custom pages** (hybrid):
- Canvas app capabilities embedded inside model-driven apps
- Best of both worlds — standard MDA navigation + custom canvas screens
- Limit to ~25 custom pages per app (performance impact beyond that)
- Use instead of embedded canvas apps for better integration and performance

### Maintainability Rules
1. **Name everything** — controls, variables, collections get meaningful names, not Button1, Label37
2. **Use components** for reusable UI patterns
3. **Limit screens per app** — decompose into multiple apps if >15 screens
4. **Avoid global variables** — use context variables and collections appropriately
5. **Use App.OnStart sparingly** — move initialization logic to named formulas
6. **Don't hardcode** — use environment variables for connection strings, URLs, feature flags
7. **Follow naming conventions** — scrXxx for screens, btnXxx for buttons, varXxx for variables, colXxx for collections
