# CMSMS Developer Handbook

The official module development documentation for [CMS Made Simple](https://www.cmsmadesimple.org), served at [developer.cmsmadesimple.org/docs](https://developer.cmsmadesimple.org/docs).

## Contributing

We welcome contributions from the community. To edit or add documentation:

1. **Fork** this repository
2. **Edit** or create `.md` files (see structure below)
3. **Update** `toc.json` if you added a new page or chapter
4. **Submit** a pull request against the `main` branch

On merge, the live site updates automatically — no manual deployment needed.

## Repository Structure

```
├── README.md              ← You are here
├── toc.json               ← Table of contents (defines navigation + metadata)
├── home.md                ← Landing page
├── intro/                 ← Chapter: Introduction
│   ├── module-development.md        ← Chapter overview
│   └── what-is-a-module.md
├── module-basics/         ← Chapter: Module Basics
│   ├── module-basics-overview.md    ← Chapter overview
│   ├── header-requirements.md
│   ├── install-uninstall-upgrade.md
│   ├── module-directories-and-paths.md
│   ├── including-a-software-license.md
│   └── best-practices.md
├── security/              ← Chapter: Module Security
│   ├── security-overview.md
│   ├── checking-permissions.md
│   ├── data-validation.md
│   ├── nonces.md
│   ├── securing-output.md
│   └── securing-input.md
├── events/                ← Chapter: Events
│   ├── events-overview.md
│   ├── sending-events.md
│   ├── handling-events.md
│   ├── core-events-reference.md
│   └── hooks.md
├── admin-interface/       ← Chapter: Admin Interface
│   ├── admin-interface-overview.md
│   ├── admin-actions.md
│   ├── admin-tabs-and-navigation.md
│   ├── admin-forms.md
│   └── ajax.md
├── smarty-tags/           ← Chapter: Smarty Tags
│   ├── smarty-tags-overview.md
│   ├── content-block-tags.md
│   ├── plugin-tags.md
│   └── tag-parameters.md
├── preferences/           ← Chapter: Preferences
│   ├── preferences-overview.md
│   ├── module-preferences-api.md
│   └── site-preferences.md
├── database/              ← Chapter: Database Operations
│   ├── database-overview.md
│   ├── using-adodb.md
│   ├── creating-tables.md
│   └── schema-management.md
├── templates/             ← Chapter: Templates
│   ├── templates-overview.md
│   ├── smarty-templates.md
│   ├── admin-templates.md
│   ├── frontend-templates.md
│   └── design-manager.md
├── users/                 ← Chapter: Users and Permissions
│   ├── users-overview.md
│   ├── roles-and-permissions.md
│   └── custom-permissions.md
├── internationalization/  ← Chapter: Internationalization
│   ├── internationalization-overview.md
│   ├── language-files.md
│   └── using-lang-strings.md
├── assets-and-resources/  ← Chapter: Assets and Resources
│   ├── assets-overview.md
│   ├── loading-css-and-javascript.md
│   └── images-and-icons.md
├── http-and-rest-apis/    ← Chapter: HTTP and REST APIs (single-page)
│   └── http-and-rest-apis.md
├── background-jobs/       ← Chapter: Background Jobs (single-page)
│   └── background-jobs.md
├── guidelines/            ← Chapter: Module Guidelines
│   ├── guidelines-overview.md
│   ├── cmsms-compliance.md
│   ├── security-rules.md
│   └── quality-rules.md
├── tutorial/              ← Chapter: Tutorial (Holidays Module)
│   ├── tutorial-overview.md
│   ├── getting-started.md
│   ├── install-and-database.md
│   ├── the-model-class.md
│   ├── admin-crud.md
│   ├── frontend-views.md
│   ├── pretty-urls.md
│   ├── search-and-more.md
│   └── starter-pack.md
├── cmsms-forge/           ← Chapter: Forge
│   ├── forge-overview.md
│   ├── submitting-your-module.md
│   ├── module-xml-packaging.md
│   └── module-info-file.md
└── developer-tools/       ← Chapter: Developer Tools
    ├── developer-tools-overview.md
    ├── module-check.md
    └── cmsms-scanner.md
```

## Writing Guidelines

- Use **standard Markdown** — the site renders it via [Parsedown](https://parsedown.org)
- Each chapter is a **directory** with an overview page (e.g. `module-basics-overview.md`)
- Single-page chapters have one `.md` file named after the directory (e.g. `background-jobs/background-jobs.md`)
- Sub-pages are individual `.md` files in the chapter directory
- Use **kebab-case** for file names: `admin-tabs-and-navigation.md`
- Start each page with a `## Heading` (h2) — this becomes the page title if not set in `toc.json`
- Use fenced code blocks with language hints:

  ````markdown
  ```php
  $db = \cms_utils::get_db();
  ```
  ````

- Smarty tags in code examples are safe — content is rendered by Parsedown, never processed by Smarty

## URL Mapping

File paths map directly to URLs on the live site:

| File | URL |
|------|-----|
| `home.md` | `developer.cmsmadesimple.org/docs/` |
| `module-basics/header-requirements.md` | `developer.cmsmadesimple.org/docs/module-basics/header-requirements` |
| `module-basics/module-basics-overview.md` | `developer.cmsmadesimple.org/docs/module-basics` |

## toc.json

The `toc.json` file controls the sidebar navigation, page titles, and meta descriptions. **You must update it when adding new pages or chapters.**

### Structure

```json
{
  "landing": {
    "title": "Module Handbook",
    "file": "home.md",
    "description": "Welcome to the CMS Made Simple Module Handbook..."
  },
  "chapters": [
    {
      "title": "Module Basics",
      "path": "module-basics",
      "description": "Covers the fundamental building blocks...",
      "pages": [
        {
          "title": "Header Requirements",
          "file": "module-basics/header-requirements.md",
          "description": "Every CMSMS module is a PHP class that extends CMSModule..."
        }
      ]
    },
    {
      "title": "Background Jobs",
      "path": "background-jobs",
      "file": "background-jobs/background-jobs.md",
      "description": "CMSMS 2.2 introduced an asynchronous job system..."
    }
  ]
}
```

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| `title` | Yes | Display title in the sidebar navigation and page `<title>` tag |
| `path` | Yes (chapters) | Directory name — also used as the URL prefix |
| `file` | Conditional | Relative path to the `.md` file. Required for single-page chapters and sub-pages |
| `pages` | Conditional | Array of sub-pages. Only for multi-page chapters |
| `description` | No | Meta description for SEO (max 160 chars). Auto-extracted from first paragraph if omitted |

### Rules

- A chapter has **either** `pages` (array) **or** `file` (string), never both
- Multi-page chapters: the overview page is listed as the first entry in `pages`
- Single-page chapters: use `file` directly, no `pages` array. The URL uses the chapter `path` (e.g. `/docs/background-jobs`)
- **Array order = display order** in the navigation sidebar

### Adding a New Page

1. Create the `.md` file in the appropriate chapter directory
2. Add an entry to the chapter's `pages` array in `toc.json`:

```json
{
  "title": "My New Page",
  "file": "module-basics/my-new-page.md",
  "description": "A brief description for search engines (max 160 chars)."
}
```

### Adding a New Chapter

**Multi-page chapter** (has sub-pages):

1. Create a new directory with an overview page
2. Add to the `chapters` array in `toc.json`:

```json
{
  "title": "My New Chapter",
  "path": "my-new-chapter",
  "description": "What this chapter covers.",
  "pages": [
    {
      "title": "Overview",
      "file": "my-new-chapter/my-new-chapter-overview.md",
      "description": "Chapter overview."
    },
    {
      "title": "First Page",
      "file": "my-new-chapter/first-page.md",
      "description": "Description of the first page."
    }
  ]
}
```

**Single-page chapter** (no sub-pages):

```json
{
  "title": "My Simple Chapter",
  "path": "my-simple-chapter",
  "file": "my-simple-chapter/my-simple-chapter.md",
  "description": "What this chapter covers."
}
```

## License

See [LICENSE](LICENSE) for details.
