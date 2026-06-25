# CMSMS Developer Documentation — Project Context

## Overview
This repository is the **CMS Made Simple Module Handbook** — the official module development documentation served at [developer.cmsmadesimple.org/docs](https://developer.cmsmadesimple.org/docs).

It is a content-only repository of Markdown files. There is no build step, no PHP runtime, and no application code. On merge to `main`, the live site updates automatically.

## Repository Structure
- `toc.json` — Table of contents controlling sidebar navigation, page titles, and meta descriptions
- `home.md` — Landing page
- Chapter directories (e.g. `module-basics/`, `security/`, `tutorial/`) each containing `.md` pages
- `README.md` — Contributor guide

## Key Rules

### File Naming
- Use **kebab-case** for all filenames: `admin-tabs-and-navigation.md`
- Multi-page chapters have an overview file named `<chapter>-overview.md`
- Single-page chapters have one file named after the directory

### Markdown Conventions
- Start each page with an `## Heading` (h2) — this becomes the page title
- Use fenced code blocks with language hints (```php, ```smarty, ```sql)
- Standard Markdown only — rendered by Parsedown, never processed by Smarty
- Smarty tags in code examples are safe (not executed)

### toc.json Rules
- A chapter has **either** `pages` (array) **or** `file` (string), never both
- Multi-page chapters: overview page is the first entry in `pages`
- Single-page chapters: use `file` directly on the chapter object
- Array order = display order in navigation
- `description` field is for SEO meta (max 160 chars)
- **Always update toc.json when adding new pages or chapters**

### URL Mapping
| File | URL |
|------|-----|
| `home.md` | `/docs/` |
| `module-basics/header-requirements.md` | `/docs/module-basics/header-requirements` |
| `module-basics/module-basics-overview.md` | `/docs/module-basics` |

## Content Domain
All documentation covers CMSMS module development:
- PHP module architecture (CMSModule class, actions, templates)
- ADODB database layer
- Smarty templating
- Security (permissions, nonces, input/output sanitization)
- Events and hooks
- Admin interface patterns
- Forge submission and packaging
- Developer tools (Module Check, CMSMS Scanner)
