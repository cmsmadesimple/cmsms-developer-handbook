## Naming Conventions

CMSMS modules share a global namespace — for module names, database tables, permissions, events, and Smarty variables. Following consistent naming conventions prevents collisions with other modules and keeps your code recognizable. This page consolidates all naming rules in one place.

### Module Name

Your module name is used as the folder name, the class name, and the identifier throughout CMSMS:

- **PascalCase** — e.g., `Holidays`, `MyGallery`, `EventCalendar`. Not `my_gallery` or `eventcalendar`.
- **Folder must match class name exactly** — if your class is `EventCalendar`, the folder must be `modules/EventCalendar/`.
- **Be specific** — avoid generic names like `Utils`, `Helper`, `Tools`, or `Manager` that could collide with other modules.
- **Don't conflict with core** — avoid names already used by CMSMS core modules (e.g., `Search`, `News`, `FileManager`).

> **Important:** The directory name, the file name, and the class name must all match exactly. CMSMS is case-sensitive — always match the case consistently.

### File Naming

| File type | Pattern | Example |
| --- | --- | --- |
| Module class | `ModuleName.module.php` | `Holidays.module.php` |
| Action files | `action.actionname.php` | `action.edit_holiday.php` |
| Lifecycle methods | `method.install.php`, `method.uninstall.php`, `method.upgrade.php` | |
| Library classes | `class.ClassName.php` | `class.HolidayItem.php` |
| Templates | `actionname.tpl` | `defaultadmin.tpl`, `edit_holiday.tpl` |
| Language files | `xx_XX.php` | `en_US.php`, `fr_FR.php` |

### Action Files

Action files follow the pattern `action.actionname.php`:

| File | Purpose |
| --- | --- |
| `action.defaultadmin.php` | Default admin action — entry point from admin navigation |
| `action.default.php` | Default frontend action — entry point from a page template call |
| `action.edit_holiday.php` | Custom action — edit a holiday record |
| `action.delete_holiday.php` | Custom action — delete a holiday record |

### Template Files

By convention, template files match their action names:

| Action file | Template file |
| --- | --- |
| `action.defaultadmin.php` | `templates/defaultadmin.tpl` |
| `action.default.php` | `templates/default.tpl` |
| `action.edit_holiday.php` | `templates/edit_holiday.tpl` |
| `action.detail.php` | `templates/detail.tpl` |

This is a convention, not a requirement — an action can load any template file.

### Database Tables

Always prefix your table names with `CMS_DB_PREFIX` followed by `mod_` and your module's short name:

```php
CMS_DB_PREFIX . 'mod_holidays'       // Main table
CMS_DB_PREFIX . 'mod_holidays_cats'  // Related table
```

Never hardcode the prefix — different installations use different prefixes.

### Permissions

CMSMS permissions are global. Generic names like `edit` or `admin` will collide with other modules. Use the pattern `Manage ModuleName` or `Use ModuleName`:

```php
// Bad
$this->CreatePermission('edit', 'Edit items');

// Good
$this->CreatePermission('Manage Holidays', 'Manage Holidays settings');
```

A common pattern is to define permission names as class constants:

```php
class Holidays extends CMSModule
{
    const MANAGE_PERM = 'manage_holidays';
    const USE_PERM    = 'use_holidays';
}
```

### Events

CMSMS events are global. The convention is `ModuleName::EventName`:

```php
// Bad
$this->CreateEvent('ItemSaved');

// Good
$this->CreateEvent('Holidays::ItemSaved');
$this->SendEvent('Holidays::ItemDeleted', $params);
```

Core events follow a **Pre/Post** pattern — `ContentEditPre` fires before the action, `ContentEditPost` fires after.

### Smarty Variables

Smarty variables in the global scope can be overwritten by other modules or core. Prefix with your module name:

```php
// Bad — global scope, will collide
$smarty->assign('items', $items);
$smarty->assign('title', $title);

// Good — prefixed
$smarty->assign('holidays_items', $items);

// Best — use CreateTemplate scope (isolated automatically)
$tpl = $smarty->CreateTemplate($this->GetTemplateResource('default.tpl'), null, null, $smarty);
$tpl->assign('items', $items);  // Safe — scoped to this template
```

### Preference Keys

Module preferences are stored in a shared table. Prefix keys with your module name:

```php
// Bad
$this->SetPreference('items_per_page', 10);

// Good
$this->SetPreference('holidays_items_per_page', 10);
```

### Language String Keys

Language keys use lowercase with underscores. Group related strings and use descriptive names:

```php
$lang['friendlyname']    = 'Holidays';
$lang['admindescription'] = 'Manage public holidays';
$lang['holiday_saved']   = 'Holiday saved successfully.';
$lang['confirm_delete']  = 'Are you sure you want to delete this holiday?';
```

The keys `friendlyname` and `admindescription` are expected by the `GetFriendlyName()` and `GetAdminDescription()` methods.

### Release Versions

The Forge enforces a strict version format for release names. Versions must follow three-part numeric versioning with an optional pre-release suffix:

**Format:** `MAJOR.MINOR.PATCH` or `MAJOR.MINOR.PATCH-suffixN`

| Valid | Invalid |
| --- | --- |
| `1.2.3` | `1.2` (missing patch) |
| `1.2.3-beta1` | `1.2.3beta1` (missing hyphen) |
| `1.2.3-alpha2` | `1.2.3-beta.1` (dot before number) |
| `1.2.3-rc1` | `1.2-beta1` (missing patch) |

Rules:

- Always use **three numeric parts** separated by dots: `MAJOR.MINOR.PATCH`
- Pre-release suffixes are optional and must be one of: `-alpha`, `-beta`, or `-rc` followed immediately by a number
- The hyphen before the suffix is required
- No dots between the suffix and its number
- Increment the version for every release, even minor fixes
- The version returned by `GetVersion()` in your module class must match the release name on the Forge

```php
// In your module class
public function GetVersion() { return '1.2.3'; }
```

### Quick Reference

| Element | Convention | Example |
| --- | --- | --- |
| Module name | PascalCase | `EventCalendar` |
| Module folder | Matches class name | `modules/EventCalendar/` |
| Action files | `action.name.php` (lowercase) | `action.edit_event.php` |
| Template files | `name.tpl` (matches action) | `edit_event.tpl` |
| Library classes | `class.ClassName.php` | `class.EventItem.php` |
| Language files | `xx_XX.php` | `en_US.php` |
| Language keys | lowercase_with_underscores | `event_saved` |
| Database tables | `CMS_DB_PREFIX.'mod_name'` | `mod_eventcalendar` |
| Permissions | `manage_modulename` | `manage_eventcalendar` |
| Events | `ModuleName::EventName` | `EventCalendar::ItemSaved` |
| Smarty variables | Prefixed or scoped | `eventcalendar_items` |
| Preference keys | Prefixed with module name | `eventcalendar_per_page` |
| Release versions | `MAJOR.MINOR.PATCH[-suffix]` | `1.2.3`, `1.2.3-beta1` |

### Next Steps

Now that you understand the naming rules, continue to [Best Practices](/docs/module-basics/best-practices) for broader coding conventions and patterns every module should follow.
