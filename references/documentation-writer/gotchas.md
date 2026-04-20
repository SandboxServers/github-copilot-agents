# Documentation Gotchas

Platform quirks, rendering differences, and technical surprises that trip up documentation authors.
Knowing these up front prevents broken docs and frustrated readers.

## Markdown Rendering Differences

### GitHub vs Azure DevOps vs Docusaurus vs SharePoint
- **Admonitions:** GitHub uses `> [!NOTE]`, `> [!WARNING]`. Docusaurus uses `:::note`, `:::warning`. ADO supports GitHub-style. SharePoint supports neither
- **Mermaid diagrams:** GitHub renders natively. ADO renders natively since 2023. Docusaurus requires a plugin. SharePoint does not support Mermaid at all
- **HTML in Markdown:** GitHub strips some HTML tags for security. Docusaurus allows full HTML. ADO strips `<style>` and `<script>` tags
- **Table alignment:** GitHub supports `|:---|:---:|---:|` alignment syntax. Some renderers ignore it entirely
- **Nested lists:** Indentation varies — GitHub expects 2 spaces, some parsers require 4. Test on your target platform
- **Task lists:** `- [ ]` checkboxes render interactively in GitHub and ADO. Other renderers show them as plain text or don't render at all

### Mermaid Diagram Support
- Mermaid version differs by platform — newer syntax (timeline, mindmap) may not render in older GitHub Enterprise versions
- Maximum diagram complexity varies — very large diagrams time out or fail to render silently
- Dark mode compatibility: some Mermaid themes have poor contrast in dark mode — test both
- **Workaround:** For critical diagrams, include a rendered PNG/SVG as a fallback alongside the Mermaid source

## Link and Path Gotchas

### Relative Links Break When Docs Move
- Moving a file to a new folder breaks every relative link pointing to or from it
- Renaming a file breaks all inbound relative references
- **Fix:** Use CI link-checking tools (`markdown-link-check`, `lychee`) to catch broken links automatically
- Prefer relative links for within-repo references — but run link checks on every PR

### Anchor Links Are Fragile
- Auto-generated heading anchors differ by platform: `## My Section` → `#my-section` (GitHub) vs `#my-section` (most others) vs `#My-Section` (some wiki engines)
- Renaming a heading breaks all anchor links — including those from external sites
- Special characters in headings produce unpredictable anchors across platforms

## Code Block Gotchas

### Language Identifiers Affect Syntax Highlighting
- Using `bash` vs `shell` vs `sh` produces different highlighting in different renderers
- `powershell` vs `pwsh` — some platforms only recognize one
- Missing language identifier: GitHub renders as plain text, Docusaurus may guess incorrectly
- **Common safe identifiers:** `bash`, `powershell`, `json`, `yaml`, `csharp`, `python`, `typescript`, `bicep`, `hcl`

### Tab vs Space Indentation in Code Blocks
- Tabs may render as 4 or 8 spaces depending on the renderer
- Copy-paste from IDE may introduce invisible tab/space inconsistencies
- YAML code blocks are particularly sensitive — wrong indentation breaks the example

## Performance and Size Gotchas

### Large Documents Cause Renderer Issues
- GitHub Markdown preview struggles with files over 1 MB or ~2000 lines
- Very long single-page docs cause slow load times and poor search indexing
- **Fix:** Split documents over 500 lines into multiple pages with clear navigation
- Use a table of contents page that links to sub-documents

### Embedded Images and Base64
- Base64-embedded images bloat file size and make git diffs unreadable
- Large images slow page load and break mobile rendering
- **Fix:** Store images as separate files in an `/images` or `/media` folder
- Use relative paths for repo-hosted images, absolute URLs only for externally hosted content
- Compress images before committing — PNG screenshots often compress 50%+ with tools like `pngquant`

## Git and Collaboration Gotchas

### Word-Wrap Changes Create Diff Noise
- Re-wrapping paragraphs at a different line length touches every line — obscures real content changes
- **Fix:** Pick a convention (one sentence per line, or soft-wrap at 120 chars) and enforce it
- One sentence per line ("semantic line breaks") produces the cleanest diffs for prose

### Binary Files in Doc Repos
- Images, PDFs, and other binaries bloat repo size over time
- Git stores full copies of binaries — no delta compression
- **Fix:** Use Git LFS for images and binaries, or host externally and link
- Prefer SVG over PNG for diagrams — SVG is text-based, diffs well, and scales without pixelation

## Sources

- [GitHub Flavored Markdown Spec](https://github.github.com/gfm/)
- [Azure DevOps Markdown guidance](https://learn.microsoft.com/azure/devops/project/wiki/markdown-guidance)
- [Docusaurus Markdown features](https://docusaurus.io/docs/markdown-features)
