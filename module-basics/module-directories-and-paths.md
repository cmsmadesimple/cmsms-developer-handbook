## Module Directories and Paths

CMSMS expects modules to follow a specific directory layout. Understanding this structure — and the API methods for referencing paths — is essential for writing portable, well-organized modules.

### Module Location

All modules live under the `modules/` directory of your CMSMS installation:

```
/home/yoursite/public_html/modules/YourModule/
```

The directory name must exactly match the module class name and the `.module.php` filename. CMSMS is case-sensitive.

### Standard Directory Structure

A fully developed module typically follows this layout:

```
modules/YourModule/
├── YourModule.module.php      # Main module class (required)
├── method.install.php         # Installation routine
├── method.uninstall.php       # Uninstallation routine
├── method.upgrade.php         # Upgrade routine
├── moduleinfo.ini             # Forge metadata
├── action.defaultadmin.php    # Default admin action
├── action.default.php         # Default frontend action
├── action.*.php               # Additional actions
├── docs/
│   ├── help.inc               # Module help (HTML fragment)
│   └── changelog.inc          # Change log (HTML fragment)
├── lang/
│   ├── en_US.php              # English language strings (required)
│   └── ext/                   # Translations from the Translation Center
│       ├── fr_FR.php
│       └── de_DE.php
├── lib/
│   ├── class.YourItem.php     # Model classes
│   └── class.YourQuery.php    # Query/repository classes
├── templates/
│   ├── defaultadmin.tpl       # Admin templates
│   ├── default.tpl            # Frontend templates
│   └── *.tpl                  # Additional templates
├── css/                       # Module stylesheets (optional)
├── images/                    # Module images/icons (optional)
└── js/                        # Module JavaScript (optional)
```

#### What goes where:

| Location | Purpose |
| --- | --- |
| Root directory | Module class, action files, lifecycle methods, moduleinfo.ini |
| `lang/` | Language files. `en_US.php` is the base. Translations go in `lang/ext/` |
| `lib/` | PHP classes — models, queries, helpers. Auto-loaded by CMSMS |
| `templates/` | Smarty template files for admin and frontend views |
| `docs/` | Help and changelog HTML fragments loaded by GetHelp() and GetChangeLog() |
| `css/`, `js/`, `images/` | Static assets. Optional — keep frontend assets minimal for distributable modules |

### File Naming Conventions

| File type | Pattern | Example |
| --- | --- | --- |
| Module class | `ModuleName.module.php` | `Holidays.module.php` |
| Action files | `action.actionname.php` | `action.edit_holiday.php` |
| Lifecycle methods | `method.install.php`, `method.uninstall.php`, `method.upgrade.php` |  |
| Library classes | `class.ClassName.php` | `class.HolidayItem.php` |
| Templates | `actionname.tpl` | `defaultadmin.tpl`, `edit_holiday.tpl` |
| Language files | `xx_XX.php` | `en_US.php`, `fr_FR.php` |

### Path API Methods

Never hardcode absolute paths. CMSMS provides methods and constants to build paths dynamically:

#### Module path (filesystem)

```php
// Returns the absolute filesystem path to your module directory
$path = $this->GetModulePath();
// e.g., /home/yoursite/public_html/modules/Holidays

// Or use __DIR__ from within the module class file
$path = __DIR__;
```

#### Module URL

```php
// Returns the URL to your module directory
$url = $this->GetModuleURLPath();
// e.g., /modules/Holidays
```

Use this when referencing static assets like CSS, JS, or images from your templates:

```
&lt;link rel="stylesheet" href="{$mod->GetModuleURLPath()}/css/admin.css" /&gt;
&lt;script src="{$mod->GetModuleURLPath()}/js/admin.js"&gt;&lt;/script&gt;
```

#### Template resources

```php
// Converts a template filename into a Smarty resource string
$resource = $this->GetTemplateResource('defaultadmin.tpl');

// Use it to create a Smarty template object
$tpl = $smarty->CreateTemplate($resource, null, null, $smarty);
```

#### CMSMS root path

```
// The absolute filesystem path to the CMSMS installation root
$root = CMS_ROOT_PATH;
// e.g., /home/yoursite/public_html

// The URL path to the CMSMS root
$url = CMS_ROOT_URL;
// e.g., https://yoursite.com
```

#### Other useful constants

| Constant | Description |
| --- | --- |
| `CMS_ROOT_PATH` | Absolute filesystem path to the CMSMS installation |
| `CMS_ROOT_URL` | Base URL of the CMSMS installation |
| `CMS_DB_PREFIX` | Database table prefix (e.g., `cms_`) |
| `CMS_VERSION` | Current CMSMS version string |
| `DIRECTORY_SEPARATOR` | PHP constant — `/` on Linux, `\` on Windows |

### Important: Paths vs URLs

Never mix filesystem paths and URLs. They serve different purposes:

- **Filesystem paths** (e.g., `CMS_ROOT_PATH`, `GetModulePath()`) — for PHP file operations like `file_exists()`, `require_once()`, `file_get_contents()`.
- **URLs** (e.g., `CMS_ROOT_URL`, `GetModuleURLPath()`) — for HTML output like `<link>`, `<script>`, `<img>` tags and browser-facing links.

```php
// Correct — filesystem path for PHP
if (file_exists($this->GetModulePath() . '/lib/class.MyHelper.php')) { ... }

// Correct — URL for HTML
&lt;img src="{$mod->GetModuleURLPath()}/images/icon.png" /&gt;

// WRONG — URL used as filesystem path
if (file_exists($this->GetModuleURLPath() . '/lib/class.MyHelper.php')) { ... }
```

### Admin and Site URLs

CMSMS provides ways to reference the admin panel URL and the site's public URL:

```
// The public site URL
$site_url = CMS_ROOT_URL;
// e.g., https://yoursite.com

// The admin panel URL (from config)
$config = \cms_config::get_instance();
$admin_url = $config['admin_url'];
// e.g., https://yoursite.com/admin
```

### Temporary and Upload Paths

Modules should never write files directly into their own directory. Use the CMSMS temporary and upload locations instead:

```
// Temporary cache directory
$tmp = TMP_CACHE_LOCATION;
// e.g., /home/yoursite/public_html/tmp/cache

// Compiled Smarty templates directory
$templates_c = TMP_TEMPLATES_C_LOCATION;
// e.g., /home/yoursite/public_html/tmp/templates_c

// Uploads directory (from config)
$config = \cms_config::get_instance();
$uploads = $config['uploads_path'];
$uploads_url = $config['uploads_url'];
```
> **Note:** If your module needs to store user-uploaded files, create a subdirectory under the uploads path named after your module (e.g., `uploads/holidays/`). Never write to the module directory itself — it may not be writable, and files there will be lost on module upgrade.

### The Autoloader

CMSMS automatically loads classes from your module's `lib/` directory. When you reference a class like `HolidayItem` in an action file, CMSMS looks for `lib/class.HolidayItem.php` and includes it automatically.

If your module uses namespaced helper classes, you can register a custom autoloader in your module constructor:

```php
public function __construct()
{
    spl_autoload_register([$this, '_autoloader']);
    parent::__construct();
}

private function _autoloader($classname)
{
    $parts = explode("\\", $classname);
    $classname = end($parts);
    $base = $this->GetModulePath() . DIRECTORY_SEPARATOR . 'lib';
    $filename = 'class.' . $classname . '.php';

    $fn = $base . DIRECTORY_SEPARATOR . $filename;
    if (file_exists($fn)) { require_once($fn); return; }

    $fn = $base . DIRECTORY_SEPARATOR . 'checks' . DIRECTORY_SEPARATOR . $filename;
    if (file_exists($fn)) { require_once($fn); }
}
```

This allows you to organize classes into subdirectories under `lib/` while still using the standard `class.ClassName.php` naming convention.

### Special Action Names

CMSMS reserves a few action names with special behavior:

| Action file | When it runs |
| --- | --- |
| `action.defaultadmin.php` | Default action for admin requests — the entry point when clicking your module in the admin navigation |
| `action.default.php` | Default action for frontend requests when no action is specified in the module call |

All other action files follow the pattern `action.youractionname.php` and are called by specifying the action name in a URL or Smarty tag.

### Next Steps

Continue to [Including a Software License](/modules/module-basics/including-a-software-license/) to learn about the licensing requirements for CMSMS modules.
