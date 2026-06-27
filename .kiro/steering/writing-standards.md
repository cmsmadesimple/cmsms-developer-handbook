---
inclusion: fileMatch
fileMatchPattern: "**/*.md"
---

# Documentation Writing Standards

## Voice and Tone
- Write for **intermediate PHP developers** who are new to CMSMS
- Be direct and practical — show working code, not theory
- Use second person ("your module", "you can") not third person
- Keep paragraphs short (3-5 sentences max)
- Lead with the "what" and "why" before the "how"

## Page Structure
1. Start with `## Page Title` (h2)
2. Opening paragraph explaining what this page covers and why it matters
3. Sections using `###` (h3) subheadings
4. Code examples with explanations
5. End with tips, warnings, or "next steps" where appropriate

## Code Examples
- Always use fenced code blocks with language hints: ```php, ```smarty, ```sql, ```json
- Show complete, working snippets — not fragments that won't run
- Include the mandatory file header (`if (!defined('CMS_VERSION')) exit;`) in action file examples
- Use realistic variable names and table names
- Add inline comments for non-obvious logic only

## Formatting Rules
- Use **bold** for UI labels, file names on first mention, and key terms
- Use `backticks` for code references inline: method names, class names, file paths, SQL keywords
- Use blockquotes (`>`) for important warnings or notes
- Use tables for reference data (parameters, return values, method lists)
- Avoid HTML in Markdown — use standard Markdown syntax only

## Cross-References
- Link to other pages using absolute paths with the `/docs/` prefix: `[Checking Permissions](/docs/security/checking-permissions)`
- For chapter overview pages, link to the chapter path: `[Module Basics](/docs/module-basics)`
- Don't use relative paths (`../security/checking-permissions.md`) — they break on the live site
- Don't use full domain URLs for internal documentation links

## What NOT to Do
- Don't use `{php}` tags in template examples (it's a security violation)
- Don't show deprecated patterns without clearly marking them as deprecated
- Don't assume the reader has read every previous chapter — provide enough context
- Don't use placeholder text like "lorem ipsum" in examples
- Don't include the full module class in every example — show only the relevant method/section
