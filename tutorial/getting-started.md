## Tutorial: Getting Started

In this first chapter, we create the module directory, the module class file with all the required virtual methods, and the language file.

### Step 1: Create the Module Directory

Create a new directory called `Holidays` inside your CMSMS `modules/` directory:

```
modules/
└── Holidays/
```
> **Note:** The directory name, the file name, and the class name must all match exactly. CMSMS is case-sensitive.

### Step 2: Create the Module Class

Create a file called `Holidays.module.php` inside the `Holidays/` directory:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;

class Holidays extends CMSModule
{
    const MANAGE_PERM = 'manage_holidays';

    public function GetVersion() { return '0.1'; }
    public function GetFriendlyName() { return $this->Lang('friendlyname'); }
    public function GetAdminDescription() { return $this->Lang('admindescription'); }
    public function IsPluginModule() { return true; }
    public function HasAdmin() { return true; }
    public function VisibleToAdminUser() { return $this->CheckPermission(self::MANAGE_PERM); }
    public function GetAuthor() { return 'Your Name'; }
    public function GetAuthorEmail() { return 'you@example.com'; }
    public function GetAdminSection() { return 'extensions'; }
    public function MinimumCMSVersion() { return '2.2.1'; }
    public function GetDependencies() { return []; }
    public function UninstallPreMessage() { return $this->Lang('ask_uninstall'); }
}
```

#### What each method does:

- `GetVersion()` — returns `'0.1'`. CMSMS compares this with the stored version to trigger upgrades.
- `GetFriendlyName()` — the name shown in the admin navigation. Uses a lang string so it can be translated.
- `IsPluginModule()` — returns `true` because our module will display content on the frontend.
- `HasAdmin()` — returns `true` because we want an admin panel.
- `VisibleToAdminUser()` — checks if the current admin has the `manage_holidays` permission.
- `MANAGE_PERM` — a class constant for the permission name, avoiding typos throughout the code.

### Step 3: Create the Language File

Create the directory `lang/` inside `Holidays/`, then create `en_US.php`:

```php
&lt;?php
$lang['friendlyname'] = 'Holidays';
$lang['admindescription'] = 'A module for managing and displaying holidays';
$lang['ask_uninstall'] = 'Are you sure you want to uninstall the Holidays module? All holiday data will be permanently deleted.';
```

We will add more strings to this file as we develop the module. Keep it open in your editor.

### Status Check

Your directory structure should now look like this:

```
modules/Holidays/
├── Holidays.module.php
└── lang/
    └── en_US.php
```

At this point, the module is already detectable by CMSMS. You can see it in Extensions > Module Manager, but it has no functionality yet — no install routine, no admin panel, no frontend output.

Next, we'll add the install and uninstall routines to create the database table and permissions. Continue to {cms\_selflink dir='next' text='Install, Uninstall, and the Database'}.
