---
inclusion: fileMatch
fileMatchPattern: "toc.json"
---

# toc.json Management Rules

## When to Update toc.json
- **Always** when adding a new page or chapter
- When renaming or moving a page file
- When changing a page's title or description
- When reordering pages within a chapter

## Structure Rules

### Multi-page chapter:
```json
{
  "title": "Chapter Title",
  "path": "chapter-directory",
  "description": "SEO description (max 160 chars).",
  "pages": [
    {
      "title": "Overview",
      "file": "chapter-directory/chapter-directory-overview.md",
      "description": "Overview description."
    },
    {
      "title": "Sub Page",
      "file": "chapter-directory/sub-page.md",
      "description": "Sub page description."
    }
  ]
}
```

### Single-page chapter:
```json
{
  "title": "Chapter Title",
  "path": "chapter-directory",
  "file": "chapter-directory/chapter-directory.md",
  "description": "SEO description (max 160 chars)."
}
```

## Validation Checklist
- [ ] Chapter has `pages` OR `file`, never both
- [ ] All `file` paths are relative and point to existing `.md` files
- [ ] Overview page is the first entry in `pages` array
- [ ] `description` is under 160 characters
- [ ] Array order matches desired navigation order
- [ ] No trailing commas in JSON
- [ ] `path` matches the actual directory name on disk
