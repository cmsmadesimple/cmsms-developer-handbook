## Tutorial: Building a Holidays Module

This tutorial walks you through building a complete CMSMS module from scratch. By the end, you will have a fully functional "Holidays" module with an admin CRUD interface, frontend summary and detail views, pretty URLs, Search integration, and AJAX previews.

The tutorial is based on the official *Introduction to Writing Modules for CMS Made Simple* by Robert Campbell, updated for CMSMS 2.2.x and PHP 8.2+.

### What You'll Build

- A module that manages holiday records (name, date, description, published status).
- An admin panel with add, edit, delete, and list functionality.
- A frontend summary view and detail view.
- SEO-friendly (pretty) URLs.
- Search module integration.
- AJAX-powered preview.

### Prerequisites

- A working CMSMS 2.2.x installation.
- PHP 8.2+ and MySQL.
- Basic knowledge of PHP, OOP, HTML, and SQL.
- Familiarity with the CMSMS admin panel.

### Chapters

1. [Getting Started](/getting-started) — Create the module skeleton, add the virtual methods, and create the lang file.
2. [Install, Uninstall, and the Database](/install-and-database) — Create the install/uninstall routines, permissions, and the holidays database table.
3. [The Model Class](/the-model-class) — Create the HolidayItem class for managing holiday records.
4. [Admin CRUD](/admin-crud) — Build the admin panel: add, list, edit, and delete holidays.
5. [Frontend Views](/frontend-views) — Create the summary and detail views for the public website.
6. [Pretty URLs and Routes](/pretty-urls) — Add SEO-friendly URLs and route registration.
7. [Search, Lazy Loading, and AJAX](/search-and-more) — Integrate with the Search module, enable lazy loading, and add AJAX previews.

### Final File Structure

```
modules/Holidays/
├── Holidays.module.php
├── action.default.php
├── action.defaultadmin.php
├── action.delete_holiday.php
├── action.detail.php
├── action.edit_holiday.php
├── method.install.php
├── method.uninstall.php
├── lang/
│   └── en_US.php
├── lib/
│   ├── class.HolidayItem.php
│   └── class.HolidayQuery.php
└── templates/
    ├── ajax_detail.tpl
    ├── default.tpl
    ├── defaultadmin.tpl
    ├── detail.tpl
    └── edit_holiday.tpl
```

Let's get started with {cms\_selflink dir='next' text='Getting Started'}.

### Starter Pack

Want to skip ahead? Download the complete Holidays module as a ready-to-install ZIP file from the [Starter Pack](/starter-pack) page. You can install it immediately or use it as a reference while following the tutorial.
