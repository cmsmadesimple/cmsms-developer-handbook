## Header Requirements

Every CMSMS module is a PHP class that extends `CMSModule`. The base class provides a set of virtual methods that you override to tell CMSMS about your module — its name, version, author, capabilities, and dependencies.

These methods are the first thing you write when creating a new module. CMSMS calls them to populate the Module Manager, build the admin navigation, check dependencies, and determine when upgrades are needed.

### The Module Class File

Your main module file must be named `ModuleName.module.php` and placed in the `modules/ModuleName/` directory. It must contain a class with the exact same name that extends `CMSModule`:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;

class Holidays extends CMSModule
{
    public function GetVersion() { return '1.0'; }
    public function GetFriendlyName() { return $this->Lang('friendlyname'); }
    public function GetAdminDescription() { return $this->Lang('admindescription'); }
    public function IsPluginModule() { return true; }
    public function HasAdmin() { return true; }
    public function VisibleToAdminUser() { return $this->CheckPermission(self::MANAGE_PERM); }
    public function GetAuthor() { return 'Your Name'; }
    public function GetAuthorEmail() { return 'you@example.com'; }
    public function MinimumCMSVersion() { return '2.2.1'; }
    public function GetDependencies() { return []; }
}
```
> **Note:** The very first line after the opening PHP tag should be a check for `CMS_VERSION`. This is a security precaution that prevents the file from being requested directly by a browser. Every PHP file in your module should include this check.

### Required Methods

While none of these methods will cause a fatal error if omitted, you should override all of them. CMSMS relies on their return values for module management, admin navigation, and upgrade detection.

#### GetName()

Returns the internal name of your module. It must match the directory name and class name exactly. You normally do not need to override this — the base class returns the class name automatically.

```
public function GetName() { return 'Holidays'; }
```

#### GetVersion()

Returns the current version number as a string. CMSMS stores the installed version in the database and compares it against this value on each request. When they differ, the upgrade routine is triggered.

The version string must be compatible with PHP's `version_compare()` function (e.g., `'1.0'`, `'1.0.1'`, `'2.1.3'`).

```
public function GetVersion() { return '1.0'; }
```

#### GetFriendlyName()

Returns the human-readable name displayed in the admin navigation and Module Manager. Use a language string so it can be translated:

```php
public function GetFriendlyName() { return $this->Lang('friendlyname'); }
```

#### GetAdminDescription()

Returns a short description of your module, displayed in the Module Manager:

```php
public function GetAdminDescription() { return $this->Lang('admindescription'); }
```

#### IsPluginModule()

Returns `true` if your module provides frontend functionality (i.e., it can be called from a page template). Returns `false` if it is an admin-only tool.

```
public function IsPluginModule() { return true; }
```

#### HasAdmin()

Returns `true` if your module has an admin panel. When true, CMSMS will add your module to the admin navigation and look for an `action.defaultadmin.php` file.

```
public function HasAdmin() { return true; }
```

#### VisibleToAdminUser()

Returns `true` if the currently logged-in admin user should see this module in the navigation. This is where you check permissions. The module must also return `true` for `HasAdmin()`.

```php
public function VisibleToAdminUser()
{
    return $this->CheckPermission(self::MANAGE_PERM);
}
```
> **Note:** Admin users who are members of the "admin" group automatically have all permissions without needing them explicitly assigned.

#### GetDependencies()

Returns an associative array of module names and their minimum required versions. CMSMS will ensure these modules are loaded before yours. Return an empty array if your module has no dependencies:

```
// No dependencies
public function GetDependencies() { return []; }

// Depends on CMSMailer 6.0+
public function GetDependencies() { return ['CMSMailer' => '6.0']; }
```
> **Note:** Each time you change the dependencies returned by `GetDependencies()`, you must also increment your module version.

#### MinimumCMSVersion()

Returns the minimum CMSMS version required for your module to operate:

```
public function MinimumCMSVersion() { return '2.2.1'; }
```

#### GetAuthor() and GetAuthorEmail()

Return the module author's name and email address. These are displayed in the Module Manager and in the module's help:

```
public function GetAuthor() { return 'Your Name'; }
public function GetAuthorEmail() { return 'you@example.com'; }
```

### Optional Methods

#### GetHelp()

Returns an HTML string containing help documentation for your module. This is displayed when clicking "Module Help" in the admin console, or from the Module Manager's help link.

A common pattern is to load the help from a separate file:

```php
public function GetHelp()
{
    $fn = __DIR__ . '/docs/help.inc';
    return (is_file($fn)) ? @file_get_contents($fn) : '';
}
```
> **Note:** Module help can be displayed from the Module Manager even if the module is not yet installed, so do not rely on relative links to images or other module assets.

#### GetChangeLog()

Returns an HTML string containing the change history for your module:

```php
public function GetChangeLog()
{
    $fn = __DIR__ . '/docs/changelog.inc';
    return (is_file($fn)) ? @file_get_contents($fn) : '';
}
```

#### UninstallPreMessage()

Returns a confirmation message displayed to the admin before uninstalling the module. Use this to warn about data loss:

```php
public function UninstallPreMessage()
{
    return $this->Lang('ask_uninstall');
}
```

#### GetAdminSection()

Returns the admin navigation section where your module should appear. Common values are `'content'`, `'layout'`, `'usersgroups'`, `'extensions'`, `'siteadmin'`, and `'myprefs'`:

```
public function GetAdminSection() { return 'extensions'; }
```

#### LazyLoadAdmin() and LazyLoadFrontend()

Return `true` to prevent your module from being loaded into memory until it is actually needed. This improves performance when many modules are installed:

```
public function LazyLoadAdmin() { return true; }
```

See the [Best Practices](/modules/module-basics/best-practices/) page for details on when lazy loading can and cannot be used.

### The Lang File

Several of the methods above reference `$this->Lang()`. This method looks up a key in your module's language file and returns the translated string.

Language files are stored in `lang/en_US.php` inside your module directory:

```php
&lt;?php
$lang['friendlyname'] = 'Holidays';
$lang['admindescription'] = 'A module for managing and displaying holidays';
$lang['ask_uninstall'] = 'Are you sure you want to uninstall the Holidays module? All holiday data will be permanently deleted.';
```

CMSMS automatically reads this file the first time `Lang()` is called. See [Internationalization](/modules/internationalization/) for full details on language files and translations.

### Complete Example

Here is a complete, minimal module class with all recommended methods:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;

class Holidays extends CMSModule
{
    const MANAGE_PERM = 'manage_holidays';

    public function GetVersion() { return '1.0'; }
    public function GetFriendlyName() { return $this->Lang('friendlyname'); }
    public function GetAdminDescription() { return $this->Lang('admindescription'); }
    public function IsPluginModule() { return true; }
    public function HasAdmin() { return true; }
    public function GetAuthor() { return 'Your Name'; }
    public function GetAuthorEmail() { return 'you@example.com'; }
    public function GetAdminSection() { return 'extensions'; }
    public function MinimumCMSVersion() { return '2.2.1'; }
    public function GetDependencies() { return []; }
    public function UninstallPreMessage() { return $this->Lang('ask_uninstall'); }

    public function VisibleToAdminUser()
    {
        return $this->CheckPermission(self::MANAGE_PERM);
    }

    public function GetHelp()
    {
        $fn = __DIR__ . '/docs/help.inc';
        return (is_file($fn)) ? @file_get_contents($fn) : '';
    }

    public function GetChangeLog()
    {
        $fn = __DIR__ . '/docs/changelog.inc';
        return (is_file($fn)) ? @file_get_contents($fn) : '';
    }
}
```

### Next Steps

With your module class defined, the next step is to create the lifecycle files that run when your module is installed, uninstalled, or upgraded. Continue to [Install / Uninstall / Upgrade Methods](/modules/module-basics/install-uninstall-upgrade/).
