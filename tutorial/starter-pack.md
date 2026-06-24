## Tutorial: Holidays Starter Pack

To help you follow along with the tutorial, we've prepared a ready-to-install starter pack containing the complete Holidays module. You can download it, extract it into your `modules/` directory, and have a working module immediately — or use it as a reference while building your own from scratch.

### Download

[⬇ Download Holidays Starter Pack (ZIP)](/uploads/Holidays.zip)

### Installation

1. Download the ZIP file.
2. Extract the `Holidays/` folder into your CMSMS `modules/` directory.
3. Log into the CMSMS admin panel.
4. Navigate to Module Manager. Find "Holidays" in the list and click "Install".

![Module Manager showing the Holidays module ready to install](/uploads/images/Holidays/screenshot-1.jpg)

5. Go to Users & Groups and assign the "Manage Holidays" permission to your admin user or group.
6. Navigate to Extensions > Holidays to start using the module.

![Admin sidebar showing Extensions > Holidays menu item](/uploads/images/Holidays/screenshot-2.jpg)

### Screenshots

#### Admin Panel

The module provides a full CRUD interface in the admin panel. Click "Create a New Holiday" to add your first entry:

![Admin panel - Add new holiday form with name, date, published, and description fields](/uploads/images/Holidays/screenshot-3.jpg)

After adding a few holidays, the admin list view shows all entries with edit and delete icons:

![Admin panel - Holiday list view with edit and delete actions](/uploads/images/Holidays/screenshot-4.jpg)

The module includes built-in help accessible from the Module Manager:

![Module help page showing usage instructions and parameters](/uploads/images/Holidays/screenshot-5.jpg)

#### Calling the Module from a Page

To display holidays on the frontend, create a content page and add the `{Holidays}` tag:

![Content Manager page editor with Holidays tag inserted](/uploads/images/Holidays/screenshot-6.jpg)

#### Frontend Output

The summary view lists all published holidays with links to detail pages and an AJAX preview option:

![Frontend summary view showing a list of holidays](/uploads/images/Holidays/screenshot-7.jpg)

Clicking a holiday name opens the detail view with a pretty URL:

![Frontend detail view showing Presidents Day with pretty URL https://cmsmadesimple.local/Holidays/1/2/Presidents-Day/](/uploads/images/Holidays/screenshot-8.jpg)

Clicking "Preview" loads a compact AJAX preview without leaving the page — it appears for 5 seconds then hides:

![Frontend summary view with AJAX preview panel showing a holiday summary](/uploads/images/Holidays/screenshot-9.jpg)

### What's Inside

The starter pack contains 18 files organized into the standard CMSMS module structure:

```
Holidays/
├── Holidays.module.php
├── action.default.php
├── action.defaultadmin.php
├── action.delete_holiday.php
├── action.detail.php
├── action.edit_holiday.php
├── method.install.php
├── method.uninstall.php
├── assets/
│   ├── icon.svg
│   └── icon-128x128.png
├── docs/
│   ├── LICENSE
│   ├── help.inc
│   └── changelog.inc
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

### File-by-File Overview

#### Core Files

| File | What it does |
| --- | --- |
| `Holidays.module.php` | The main module class. Extends CMSModule with all required metadata methods, permission constant, frontend parameter registration, pretty URL routing, and lazy loading. |
| `method.install.php` | Creates the `manage_holidays` permission and the `mod_holidays` database table using the ADODB DataDictionary. |
| `method.uninstall.php` | Removes the permission and drops the database table. Leaves the system clean. |

#### Admin Actions

| File | What it does |
| --- | --- |
| `action.defaultadmin.php` | The admin panel entry point. Loads all holidays via HolidayQuery and displays them in a list table with edit and delete links. |
| `action.edit_holiday.php` | Handles both adding new holidays and editing existing ones. Processes the form submission with the PRG (Post-Redirect-Get) pattern. |
| `action.delete_holiday.php` | Deletes a holiday by ID and redirects back to the admin list with a confirmation message. |

#### Frontend Actions

| File | What it does |
| --- | --- |
| `action.default.php` | The frontend summary view. Shows published holidays with links to detail pages. Supports `pagelimit` and `detailpage` parameters. |
| `action.detail.php` | The frontend detail view. Displays a single holiday. Supports an alternate `detailtemplate` parameter for AJAX previews. |

#### Model Classes (lib)

| File | What it does |
| --- | --- |
| `class.HolidayItem.php` | Represents a single holiday record. Handles insert, update, delete, validation, and loading by ID. Uses parameterized queries throughout. |
| `class.HolidayQuery.php` | Extends CmsDbQueryBase to query holidays with pagination and filtering. Returns an array of HolidayItem objects via GetMatches(). |

#### Templates (templates)

| File | What it does |
| --- | --- |
| `defaultadmin.tpl` | Admin list view with "Add new" link, data table, edit/delete icons, and JavaScript delete confirmation. |
| `edit_holiday.tpl` | Admin edit form with name, date, published dropdown, and WYSIWYG description field. |
| `default.tpl` | Frontend summary listing published holidays with detail links and AJAX preview. |
| `detail.tpl` | Frontend detail view showing a single holiday with canonical URL support. |
| `ajax_detail.tpl` | Compact detail template used for AJAX preview — strips HTML tags and summarizes the description. |

#### Other Files

| File | What it does |
| --- | --- |
| `lang/en_US.php` | English language strings — 18 keys covering labels, messages, errors, and parameter help. |
| `docs/help.inc` | Module help displayed in the admin panel (Extensions > Module Manager > Help). |
| `docs/changelog.inc` | Version history displayed in the module help. |
| `docs/LICENSE` | GPL v3 license reference. |

### What This Module Demonstrates

- Standard CMSMS module structure and naming conventions.
- All required CMSModule metadata methods.
- Permission creation, checking, and cleanup.
- Database table creation with the ADODB DataDictionary.
- MVC pattern — model classes in lib/, actions as controllers, Smarty templates as views.
- Admin CRUD with the PRG pattern.
- Frontend summary and detail views with parameter registration.
- Pretty URL generation and route registration.
- AJAX preview using showtemplate=false.
- Language strings for all user-facing text.
- Parameterized SQL queries throughout.
- License header in every PHP file.
- Lazy loading on admin requests.

### Using It as a Starting Point

To use this module as a template for your own module:

1. Copy the `Holidays/` directory and rename it to your module name (e.g., `Products/`).
2. Rename `Holidays.module.php` to `Products.module.php`.
3. Change the class name from `Holidays` to `Products` inside the file.
4. Update the permission constant, table names, and lang strings.
5. Modify the HolidayItem model to match your data structure.
6. Update the templates to match your fields.

For a detailed walkthrough of how each file was built, follow the {cms\_selflink dir='prev' text='tutorial chapters'}.
