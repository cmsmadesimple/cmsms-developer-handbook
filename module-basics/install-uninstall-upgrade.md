## Install / Uninstall / Upgrade Methods

CMSMS modules have three lifecycle files that run at specific points: installation, uninstallation, and upgrade. These files are placed in the root of your module directory and are executed within the scope of your module class — meaning `$this` refers to your module instance and you have access to all its methods.

### method.install.php

This file runs once when an admin installs your module via Extensions > Module Manager. Use it to create database tables, permissions, preferences, events, and any default data your module needs.

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;

// Create a permission
$this->CreatePermission(Holidays::MANAGE_PERM, 'Manage Holidays');

// Create a database table
$db = $this->GetDb();
$dict = NewDataDictionary($db);
$taboptarray = array('mysql' => 'TYPE=MyISAM');

$flds = "
    id I KEY AUTO,
    name C(255) KEY NOTNULL,
    description X,
    published I1,
    the_date I NOTNULL
";

$sqlarray = $dict->CreateTableSQL(CMS_DB_PREFIX . 'mod_holidays', $flds, $taboptarray);
$dict->ExecuteSQLArray($sqlarray);
```

#### What you typically do in method.install.php:

- **Create permissions** — using `CreatePermission(name, description)`.
- **Create database tables** — using the ADODB DataDictionary (see [Database Operations](/modules/database/)).
- **Set default preferences** — using `$this->SetPreference('key', 'value')`.
- **Create events** — using `$this->CreateEvent('EventName')`.
- **Register event handlers** — using `$this->AddEventHandler('ModuleName', 'EventName')`.
- **Insert default data** — using direct SQL queries.

> **Note:** CMSMS uses a modified version of adodb-lite for database abstraction. The `GetDb()` method returns a reference to the single database connection created for each request. The DataDictionary provides a database-agnostic way to create and modify tables.

#### Permissions

Define permission names as class constants to avoid typos:

```php
// In your module class
const MANAGE_PERM = 'manage_holidays';

// In method.install.php
$this->CreatePermission(Holidays::MANAGE_PERM, 'Manage Holidays');
```
> **Note:** CMSMS does not have a mechanism for localizing permission names, so the human-readable string should usually be in English.

### method.uninstall.php

This file runs when an admin uninstalls your module. It must remove everything your module created — tables, permissions, preferences, events, event handlers, and any other data. The goal is to leave the system in the same state as before the module was installed.

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;

// Remove the permission
$this->RemovePermission(Holidays::MANAGE_PERM);

// Drop the database table
$db = $this->GetDb();
$dict = NewDataDictionary($db);
$sqlarray = $dict->DropTableSQL(CMS_DB_PREFIX . 'mod_holidays');
$dict->ExecuteSQLArray($sqlarray);
```

#### What you typically do in method.uninstall.php:

- **Remove permissions** — using `RemovePermission(name)`.
- **Drop database tables** — using the DataDictionary.
- **Remove preferences** — using `$this->RemovePreference('key')`.
- **Remove events** — using `$this->DeleteEvent('EventName')`.
- **Remove event handlers** — using `$this->RemoveEventHandler('ModuleName', 'EventName')`.

> **Note:** A clean uninstall is important. It allows a module to be installed, tested, and uninstalled multiple times without polluting the database with leftover tables, templates, or preferences that the admin would need to clean up manually.

#### UninstallPreMessage()

Pair your uninstall routine with a confirmation message in your module class so the admin is warned before data is deleted:

```php
public function UninstallPreMessage()
{
    return $this->Lang('ask_uninstall');
}
```

### method.upgrade.php

This file runs when CMSMS detects that the version returned by `GetVersion()` is newer than the version stored in the database. It receives the old version number as a variable, allowing you to apply incremental changes:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;

$db = $this->GetDb();

if (version_compare($oldversion, '1.1', '&lt;')) {
    // Add a new column introduced in version 1.1
    $dict = NewDataDictionary($db);
    $sqlarray = $dict->AddColumnSQL(
        CMS_DB_PREFIX . 'mod_holidays',
        'category C(100)'
    );
    $dict->ExecuteSQLArray($sqlarray);
}

if (version_compare($oldversion, '1.2', '&lt;')) {
    // Create a new permission introduced in version 1.2
    $this->CreatePermission('view_holidays', 'View Holidays');
}

if (version_compare($oldversion, '2.0', '&lt;')) {
    // Major schema change in version 2.0
    $db->Execute('ALTER TABLE ' . CMS_DB_PREFIX . 'mod_holidays ADD COLUMN slug C(255)');
    $this->CreateEvent('HolidayAdded');
}
```

#### Key points about upgrades:

- The `$oldversion` variable contains the previously installed version number.
- Use `version_compare()` to apply changes incrementally — each block handles the migration from one version to the next.
- Never assume the upgrade is from the immediately previous version. A user might upgrade from 1.0 directly to 2.0, so all intermediate migrations must run in sequence.
- After the upgrade file runs successfully, CMSMS updates the stored version to match `GetVersion()`.

### Scope and Context

All three lifecycle files are executed within the scope of your module class:

- `$this` — refers to an instance of your module class.
- `$this->GetDb()` — returns the global database connection.
- `$this->CreatePermission()`, `$this->SetPreference()`, etc. — all module API methods are available.
- Classes in your `lib/` directory are automatically loaded by the CMSMS autoloader.

### Checklist

| Task | Install | Uninstall | Upgrade |
| --- | --- | --- | --- |
| Create / drop database tables | ✓ | ✓ | ✓ (alter) |
| Create / remove permissions | ✓ | ✓ | ✓ (new perms) |
| Set / remove preferences | ✓ | ✓ | ✓ (new prefs) |
| Create / remove events | ✓ | ✓ | ✓ (new events) |
| Register / remove event handlers | ✓ | ✓ | ✓ |
| Migrate data |  |  | ✓ |

### Next Steps

Now that you understand the module lifecycle, continue to [Best Practices](/modules/module-basics/best-practices/) to learn the coding conventions and patterns every module should follow.
