# Documentation Platforms

Different platforms serve different audiences and use cases. Choosing the right platform is as important as writing the content. Put documentation where people already look — don't make them hunt for it across five systems.

## Platform Comparison

| Platform | Best For | Strengths | Weaknesses |
|---|---|---|---|
| **README.md** | Repo entry point, quick start | Lives with the code, always current, version-controlled | Limited to one file, no hierarchy, no search |
| **Wiki (ADO/GitHub)** | Team knowledge base, runbooks | Hierarchical pages, easy editing, good for ops docs | Can become a graveyard if not maintained, weak search |
| **Docusaurus / MkDocs** | External docs, API docs, public sites | Versioning, search, sidebar navigation, theming | Requires build/deploy pipeline, higher setup cost |
| **SharePoint** | Enterprise audiences, non-dev stakeholders | Familiar to business users, permissions, integrations | Poor markdown support, no code blocks, clunky editing |
| **Confluence** | Cross-team collaboration, project docs | Templates, macros, integrations, space-based organization | Partial markdown, sprawls without governance, paid |

## Platform-Specific Guidance

### README.md
Maximum 2 screens of scrolling. Focus on project description, quick start, and links to deeper documentation. Include badges for build status, coverage, and latest version. The README is the front door — keep it inviting and current.

### Wiki (ADO/GitHub)
Organize by audience (ops, dev, architecture) not by technology. Use a table of contents on the landing page. Assign owners to sections and review quarterly. Delete or archive pages that haven't been updated in 6 months.

### Docusaurus / MkDocs
Leverage sidebars for navigation structure. Use admonitions (tip, warning, note) for callouts. Enable versioning for API documentation. Set up a CI pipeline to build and deploy on merge. Treat docs like code — PRs, reviews, and automated link checking.

### SharePoint
Keep it simple. No code blocks — link to technical docs in the repo or wiki instead. Use visuals and short paragraphs. Best for executive summaries, org-wide policies, and non-technical procedures.

### Confluence
Use templates for consistency across spaces. Create one space per project or team. Archive aggressively — stale pages in Confluence are worse than no pages. Set page expiry reminders for time-sensitive content.

## Docs-as-Code Principles

Treat documentation with the same rigor as source code:

- **Version control** — store docs in the repo alongside the code they describe
- **Pull requests** — documentation changes go through review, just like code
- **Automated checks** — lint markdown, check links, validate formatting in CI
- **Single source of truth** — don't duplicate content across platforms. Write once, link everywhere
- **Tests** — verify code examples compile and run. Broken examples destroy trust
- **Deploy pipeline** — docs site builds and deploys automatically on merge

## Formatting Standards

### Markdown Conventions
- **Headings**: ATX style (`#`), sentence case, hierarchical — never skip levels
- **Lists**: `-` for unordered, `1.` for ordered. Consistent indentation (2 or 4 spaces, pick one)
- **Code**: Inline with single backticks, blocks with triple backticks plus language identifier
- **Links**: Descriptive text always. `[Configure RLS](./rls-guide.md)` not `[click here](./rls-guide.md)`
- **Tables**: Comparisons and structured data. Keep columns under 5. Align for readability
- **Images**: Alt text required. Prefer diagrams over screenshots. Store in `/media` or `/images`
- **Admonitions**: `> [!NOTE]` / `> [!WARNING]` in GitHub/ADO. `:::tip` / `:::warning` in Docusaurus

### Naming Conventions
- Files: kebab-case — `getting-started.md`, `adr-042-cosmos-selection.md`
- Folders: kebab-case — `how-to-guides/`, `architecture-decisions/`
- Images: descriptive kebab-case — `network-topology-hub-spoke.png`
- No spaces in filenames — ever

### Diagram Standards
- Use Mermaid for simple diagrams — supported in GitHub, ADO, and Docusaurus
- Use draw.io for complex architecture diagrams — export as SVG and store the source file
- Include the diagram source alongside the rendered image for future editing
- Label everything — boxes, arrows, data flows, protocols
- Date diagrams — include "Last updated: YYYY-MM" in the diagram or caption
