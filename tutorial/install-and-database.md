## Tutorial: Install, Uninstall, and the Database

Our module needs a database table to store holidays and a permission to control admin access. We create these in the install routine and remove them in the uninstall routine.

### Step 1: Create method.install.php

Create `method.install.php` in the `Holidays/` directory:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;

// Create the permission
$this->CreatePermission(Holidays::MANAGE_PERM, 'Manage Holidays');

// Create the database table
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

#### What this does:

- Creates a permission called `manage_holidays` that admins can assign to users/groups.
- Creates a table called `cms_mod_holidays` (prefixed with `CMS_DB_PREFIX`) with columns for id, name, description, published flag, and date (stored as a unix timestamp).
- The DataDictionary handles the SQL generation — `I KEY AUTO` means auto-incrementing integer primary key, `C(255)` means VARCHAR(255), `X` means TEXT, `I1` means TINYINT.

### Step 2: Create method.uninstall.php

Create `method.uninstall.php` in the `Holidays/` directory:

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

This removes everything the install created, leaving the system clean.

### Step 3: Add Lang Strings

We already added `ask_uninstall` in the previous chapter. The `UninstallPreMessage()` method in our module class uses this string to confirm uninstallation.

### Step 4: Install the Module

1. Navigate to Extensions > Module Manager in the CMSMS admin.
2. Find "Holidays" in the list and click "Install".
3. The module should install successfully, creating the permission and database table.

You should now see "Holidays" in the admin navigation under Extensions. Clicking it will show a blank page — we haven't created the admin action yet.

Go to Users & Groups and ensure your admin user (or group) has the "Manage Holidays" permission assigned.

### Status Check

```
modules/Holidays/
├── Holidays.module.php
├── method.install.php
├── method.uninstall.php
└── lang/
    └── en_US.php
```

The module installs and uninstalls cleanly. Next, we'll create the model class for working with holiday records. Continue to {cms\_selflink dir='next' text='The Model Class'}.
