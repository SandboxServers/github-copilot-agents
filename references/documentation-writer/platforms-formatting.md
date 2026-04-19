## Documentation Platforms and Adaptation

### Where Docs Live

| Platform | Best For | Markdown Support | Structure | Notes |
|---|---|---|---|---|
| **README.md** | Repo entry point | Full GitHub/ADO Markdown | Flat (per repo) | Keep concise, link to deeper docs |
| **Wiki (ADO/GitHub)** | Team knowledge base | ADO/GitHub Markdown + extras | Hierarchical pages | Good for runbooks, onboarding, architecture |
| **Docusaurus** | External/public docs | MDX (Markdown + JSX) | Sidebar-driven | Versioned docs, search, theming |
| **SharePoint** | Enterprise/non-dev audiences | Limited | Page-based | Good for exec summaries, org-wide policies |
| **Confluence** | Cross-team collaboration | Partial (native editor) | Space → Page hierarchy | Templates, macros, integrations |

### Adaptation Rules
- **README.md**: Max 2 screens of content. Project focus, getting started, links to deeper docs. Include badges for build status, coverage.
- **Wiki**: Organize by audience (ops, dev, architecture) not by technology. Use table of contents.
- **Docusaurus**: Leverage sidebars for navigation, use admonitions (tip/warning/note), enable versioning for APIs.
- **SharePoint**: Keep it simple. No code blocks. Use visuals. Link to technical docs elsewhere.
- **Confluence**: Use templates for consistency. Create a space per project. Archive aggressively.

## Formatting Standards

### Markdown Conventions
1. **Headings**: ATX style (`#`), sentence case, hierarchical (don't skip levels)
2. **Lists**: Use `-` for unordered, `1.` for ordered. Consistent indentation (2 or 4 spaces, pick one)
3. **Code**: Inline with single backticks, blocks with triple backticks + language identifier
4. **Links**: Descriptive text, never "click here". `[Configure RLS](./rls-guide.md)` not `[link](./rls-guide.md)`
5. **Tables**: Use for comparisons and structured data. Keep columns narrow. Align for readability.
6. **Images**: Alt text always. Prefer diagrams over screenshots (diagrams don't go stale). Store images in a `/media` or `/images` folder alongside docs.
7. **Admonitions**: Use `> [!NOTE]`, `> [!WARNING]`, `> [!IMPORTANT]` in ADO/GitHub. Use `:::tip` / `:::warning` in Docusaurus.

### Naming Conventions
- Files: kebab-case (`getting-started.md`, `adr-042-cosmos-selection.md`)
- Folders: kebab-case (`how-to-guides/`, `architecture-decisions/`)
- Images: descriptive kebab-case (`network-topology-hub-spoke.png`)
- No spaces in filenames — ever

### Diagram Standards
- Use Mermaid for simple diagrams (supported in GitHub, ADO, Docusaurus)
- Use draw.io/diagrams.net for complex architecture diagrams (export as SVG + store source)
- Include the diagram source alongside the rendered image
- Label everything — boxes, arrows, data flows
- Date diagrams — put "Last updated: YYYY-MM" in the diagram or caption
