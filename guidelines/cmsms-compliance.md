## CMSMS Compliance Guidelines

These rules ensure your module follows CMS Made Simple conventions for naming, structure, API usage, and compatibility. Violations can cause conflicts with other modules, break on different CMSMS installations, or use deprecated APIs that will be removed in future versions.

These guidelines are enforced by [Module Check](/module-check) and the [CMSMS Scanner](/cmsms-scanner). Each rule includes a severity level, explanation, and code examples.

### CMSMS\_CMS\_001: Module table without module\_ prefix after CMS\_DB\_PREFIX

**Severity:** medium

CMSMS reserves unprefixed table names for core. Module tables must use module\_<modulename>\_ to avoid collisions and make ownership clear.

**Fix:** Rename tables to module\_<yourmodule>\_<tablename>.

```sql
// Bad
$db-&gt;GetAll('SELECT * FROM ' . CMS_DB_PREFIX . 'custom_items');
```
```sql
// Good
$db-&gt;GetAll('SELECT * FROM ' . CMS_DB_PREFIX . 'module_mymod_items');
```

### CMSMS\_CMS\_002: Hardcoded absolute path instead of CMS path constants

**Severity:** medium

Hardcoded paths break when the site moves to a different server, directory, or hosting environment. CMSMS provides CMS\_ROOT\_PATH and cms\_join\_path() for this.

**Fix:** Use cms\_join\_path(CMS\_ROOT\_PATH, 'modules', ...) or $config['uploads\_path'].

```
// Bad
$path = '/var/www/html/modules/MyModule/data/cache.json';
```
```php
// Good
$path = cms_join_path($this-&gt;GetModulePath(), 'data', 'cache.json');
```

### CMSMS\_CMS\_003: Hardcoded user-facing string in SetError/SetMessage

**Severity:** low

Hardcoded English strings in UI methods prevent localization and make string management inconsistent. CMSMS provides $this->Lang() backed by lang/ files.

**Fix:** Add the string to lang/en\_US.php and reference via $this->Lang('key').

```php
// Bad
$this-&gt;SetError('The item could not be saved to the database.');
```
```php
// Good
$this-&gt;SetError($this-&gt;Lang('error_item_save_failed'));
```

### CMSMS\_CMS\_004: Deprecated global $gCms for database access

**Severity:** medium

The global $gCms pattern is deprecated since CMSMS 2.0. It pollutes scope and breaks in future versions. Use cmsms() or CmsApp::get\_instance().

**Fix:** Replace with $db = cmsms()->GetDb();

```
// Bad
global $gCms;
$db = $gCms-&gt;GetDb();
$config = $gCms-&gt;GetConfig();
```
```
// Good
$db = cmsms()-&gt;GetDb();
$config = cmsms()-&gt;GetConfig();
```

### CMSMS\_CMS\_005: echo ProcessTemplate() — deprecated output pattern

**Severity:** low

echo $this->ProcessTemplate() is the CMSMS 1.x pattern. Modern CMSMS uses ProcessTemplateFromDatabase() or Smarty's fetch() method.

**Fix:** Use $smarty->CreateTemplate() + fetch(), or ProcessTemplateFromDatabase().

```php
// Bad
echo $this-&gt;ProcessTemplate('mytemplate.tpl');
```
```php
// Good
$tpl = $smarty-&gt;CreateTemplate($this-&gt;GetTemplateResource('mytemplate.tpl'));
echo $tpl-&gt;fetch();
```

### CMSMS\_CMS\_006: Admin action file missing CheckPermission() call

**Severity:** high

Every admin action must gate on CheckPermission() before performing any operation. Without it, any authenticated admin user — regardless of role — can access the functionality.

**Fix:** Add CheckPermission() at the top of the action file, before any logic.

```sql
// Bad
// action.admin_settings.php
$db = cmsms()-&gt;GetDb();
$rows = $db-&gt;GetAll('SELECT * FROM ' . CMS_DB_PREFIX . 'module_mymod_config');
```
```php
// Good
// action.admin_settings.php
if (!$this-&gt;CheckPermission('Manage MyModule')) {
    echo $this-&gt;ShowErrors($this-&gt;Lang('error_permission'));
    return;
}
$db = cmsms()-&gt;GetDb();
```

### CMSMS\_CMS\_007: Direct HTML output in action file instead of Smarty template

**Severity:** medium

CMSMS modules must use Smarty templates for HTML output. Direct echo of HTML in action files prevents admin theme integration and makes the UI unmaintainable.

**Fix:** Move HTML to a .tpl file. Assign data via $smarty->assign() and render with fetch().

```html
// Bad
echo '&lt;div class="pagecontainer"&gt;&lt;table&gt;&lt;tr&gt;&lt;td&gt;' . $name . '&lt;/td&gt;&lt;/tr&gt;&lt;/table&gt;&lt;/div&gt;';
```
```
// Good
$tpl-&gt;assign('name', $name);
echo $tpl-&gt;fetch();
```

### CMSMS\_CMS\_008: SQL query with hardcoded table name missing CMS\_DB\_PREFIX

**Severity:** medium

CMSMS allows custom table prefixes. Hardcoding 'module\_' or 'cms\_' directly in SQL breaks on any installation that changed the default prefix.

**Fix:** Always use CMS\_DB\_PREFIX . 'table\_name' or cms\_db\_prefix() . 'table\_name'.

```sql
// Bad
$db-&gt;GetAll("SELECT * FROM module_news_items WHERE active = 1");
```
```sql
// Good
$db-&gt;GetAll('SELECT * FROM ' . CMS_DB_PREFIX . 'module_news_items WHERE active = 1');
```

### CMSMS\_CMS\_009: Raw HTML <form> in frontend template instead of {form\_start}

**Severity:** medium

Raw <form> tags bypass CMSMS's action routing and CSRF protection. {form\_start} automatically includes the module action, return ID, and CSRF token.

**Fix:** Replace <form> with {form\_start action='myaction'} and </form> with {form\_end}.

```html
// Bad
&lt;form action="moduleinterface.php" method="post"&gt;
  &lt;input type="hidden" name="mact" value="MyModule,cntnt01,submit,0"&gt;
&lt;/form&gt;
```
```smarty
// Good
{form_start action='submit'}
  {* fields *}
{form_end}
```

### CMSMS\_CMS\_010: Module uses another module without GetDependencies()

**Severity:** low

Loading another module at runtime without declaring the dependency in GetDependencies() means CMSMS can't enforce load order or warn when the dependency is missing.

**Fix:** Add the module to GetDependencies() with its minimum required version.

```
// Bad
$news = cms_utils::get_module('News');
$news-&gt;DoSomething();
```
```php
// Good
// In module class:
public function GetDependencies() { return ['News' =&gt; '2.50']; }
// Then:
$news = cms_utils::get_module('News');
```

### CMSMS\_CMS\_012: Smarty {php} tag in template

**Severity:** high

{php} tags execute arbitrary PHP in the template layer. They're deprecated in Smarty 3, removed in Smarty 4/5, and are a security/maintenance liability.

**Fix:** Move all logic to the action file or a module method. Pass computed values to the template via $smarty->assign().

```sql
// Bad
{php}
  $db = cmsms()-&gt;GetDb();
  echo $db-&gt;GetOne('SELECT COUNT(*) FROM items');
{/php}
```
```html
// Good
{* Assigned in action: $tpl-&gt;assign('count', $count); *}
&lt;p&gt;Total: {$count}&lt;/p&gt;
```

### CMSMS\_CMS\_013: Generic class filename in lib/ without module prefix

**Severity:** medium

CMSMS modules share a global autoloader namespace. A class named 'Helper' or 'Utils' in one module will collide with the same name in another. Prefix with the module name: class.MyModule\_Utils.php or use a PHP namespace.

**Fix:** Rename the class file to include the module name: class.MyModule\_Helper.php, or use a PHP namespace matching the module.

```
// Bad
lib/class.helper.php
lib/class.utils.php
lib/class.api.php
```
```
// Good
lib/class.News_helper.php
lib/class.news_ops.php
lib/class.Gallery_utils.php
```

### CMSMS\_CMS\_014: Preference key without module name prefix

**Severity:** low

CMSMS stores all module preferences in a shared table. Short generic keys like 'enabled', 'mode', 'limit' can collide across modules. Prefix with the module name or a unique abbreviation.

**Fix:** Prefix preference keys: $this->SetPreference('mymodule\_enabled', '1') instead of $this->SetPreference('enabled', '1').

```php
// Bad
$this-&gt;SetPreference('enabled', '1');
$this-&gt;GetPreference('mode', 'default');
```
```php
// Good
$this-&gt;SetPreference('logwatch_enabled', '1');
$this-&gt;GetPreference('logwatch_mode', 'default');
```

### CMSMS\_CMS\_015: Permission name without module identifier

**Severity:** medium

CMSMS permissions are global. A permission named 'edit' or 'admin' will collide. The convention is 'Manage ModuleName' or 'Use ModuleName'.

**Fix:** Use the pattern 'Manage ModuleName' or 'Use ModuleName' for permission names.

```
// Bad
CreatePermission('edit', 'Edit items');
CheckPermission('admin');
```
```
// Good
CreatePermission('Manage MyModule', 'Manage MyModule settings');
CheckPermission('Manage MyModule');
```

### CMSMS\_CMS\_016: Event name without module prefix

**Severity:** low

CMSMS events are global. The convention is ModuleName::EventName (e.g. 'News::NewsArticleAdded'). Unprefixed events like 'ItemSaved' will collide.

**Fix:** Use the ModuleName::EventName pattern for all custom events.

```php
// Bad
$this-&gt;CreateEvent('ItemSaved');
$this-&gt;SendEvent('ItemDeleted', $params);
```
```php
// Good
$this-&gt;CreateEvent('MyModule::ItemSaved');
$this-&gt;SendEvent('MyModule::ItemDeleted', $params);
```

### CMSMS\_CMS\_017: Smarty variable assigned without module prefix

**Severity:** low

Smarty variables are global scope. Generic names like 'items', 'data', 'title' will be overwritten by other modules or core. Prefix with the module name or action ID.

**Fix:** Prefix Smarty variables: $smarty->assign('mymodule\_items', $items) or use the $id action prefix.

```
// Bad
$smarty-&gt;assign('items', $items);
$smarty-&gt;assign('title', $title);
```
```
// Good
$smarty-&gt;assign('news_items', $items);
$tpl-&gt;assign('items', $items);  // CreateTemplate scope is isolated
```

### CMSMS\_CMS\_018: Error reporting or display\_errors set in production code

**Severity:** medium

Setting error\_reporting() or display\_errors permanently in a module overrides the site administrator's configuration. If another developer tries to debug their site, your module's settings will interfere with their output. Error reporting configuration belongs in php.ini or the CMS config, not in module code.

**Fix:** Remove error\_reporting() and ini\_set('display\_errors') calls. If you need error handling, use a custom error handler that respects the site's existing configuration.

```
// Bad
error_reporting(E_ALL);
ini_set('display_errors', '1');
```
```
// Good
// Let the site admin control error reporting via php.ini or config.php
// If you need custom error handling:
set_error_handler(function($errno, $errstr, $errfile, $errline) {
    // log to module-specific log, don't change global settings
});
```

### CMSMS\_CMS\_019: Undocumented use of external service

**Severity:** medium

Modules that connect to external services must clearly disclose this in the module's readme.md file. This is required for legal compliance (GDPR, privacy laws) and user trust. For each external service the module uses, the readme.md must include:
1. What the service is and what it is used for.
2. What data is sent to the service and under what conditions (e.g. on install, on every page load, on admin action).
3. A link to the service's Terms of Use / Terms of Service.
4. A link to the service's Privacy Policy.
This applies to all external connections including API calls, license verification, update checks, analytics, CDN resources, and any data transmission to third-party servers.

**Fix:** Add a 'Third-Party Services' section to your readme.md documenting each external service. Example:
== Third-Party Services ==
This module connects to the following external services:
= Example API (https://api.example.com) =
\* \*\*Purpose:\*\* Used to verify license keys and check for module updates.
\* \*\*Data sent:\*\* Your license key and site URL are transmitted when you activate the module and during daily update checks.
\* \*\*Terms of Service:\*\* https://example.com/terms
\* \*\*Privacy Policy:\*\* https://example.com/privacy

```
// Bad
$response = file_get_contents('https://api.example.com/check?key=' . $license_key);
```
```
// Good
// In readme.md:
// == Third-Party Services ==
// = Example API (https://api.example.com) =
// * Purpose: License verification and update checks.
// * Data sent: License key and site URL on activation and daily checks.
// * Terms of Service: https://example.com/terms
// * Privacy Policy: https://example.com/privacy
$response = file_get_contents('https://api.example.com/check?key=' . $license_key);
```

### CMSMS\_CMS\_020: Missing recommended module metadata method

**Severity:** medium

CMSMS modules should implement metadata methods so the CMS can properly display, manage, and validate the module. Core methods (GetName, GetVersion, GetFriendlyName, GetAuthor, GetAuthorEmail) are essential for identification. Functionality methods (HasAdmin, IsPluginModule, GetDependencies, GetHelp, VisibleToAdminUser) control behavior. Missing GetAuthor() means the module has no identified contributor.

**Fix:** Implement the missing metadata methods in your main module class. At minimum: GetName(), GetVersion(), GetFriendlyName(), GetAuthor(), GetAuthorEmail(), GetAdminDescription(), HasAdmin(), IsPluginModule(), MinimumCMSVersion(), GetHelp(), VisibleToAdminUser().

```
// Bad
class MyModule extends CMSModule {
    function GetName() { return 'MyModule'; }
    function GetVersion() { return '1.0'; }
    // Missing: GetAuthor, GetAuthorEmail, GetHelp, etc.
}
```
```php
// Good
class MyModule extends CMSModule {
    function GetName() { return 'MyModule'; }
    function GetVersion() { return '1.0'; }
    function GetFriendlyName() { return $this-&gt;Lang('friendlyname'); }
    function GetAuthor() { return 'Your Name'; }
    function GetAuthorEmail() { return 'you@example.com'; }
    function GetAdminDescription() { return $this-&gt;Lang('description'); }
    function HasAdmin() { return true; }
    function IsPluginModule() { return false; }
    function MinimumCMSVersion() { return '2.2.0'; }
    function GetHelp() { /* ... */ }
    function VisibleToAdminUser() { return $this-&gt;CheckPermission('Manage MyModule'); }
    function GetDependencies() { return []; }
}
```

### CMSMS\_CMS\_021: Debug output statement left in production code

**Severity:** medium

Debug output functions dump internal state (variables, stack traces, object structures) directly to the browser. In production, this exposes sensitive information like database credentials, file paths, and internal logic to end users. These calls should be removed or wrapped in a debug-mode check before release.

**Fix:** Remove all var\_dump(), print\_r(), and debug\_print\_backtrace() calls. If you need runtime debugging, use error\_log() to write to the server log instead, or wrap in a debug-mode check: if (defined('CMS\_DEBUG') && CMS\_DEBUG) { var\_dump($data); }

```
// Bad
var_dump($params);
print_r($db-&gt;ErrorMsg());
debug_print_backtrace();
```
```
// Good
error_log('MyModule debug: ' . print_r($params, true));
// Or wrap in debug check:
if (defined('CMS_DEBUG') &amp;&amp; CMS_DEBUG) {
    error_log(print_r($params, true));
}
```

### CMSMS\_CMS\_022: Module folder structure missing required files or directories

**Severity:** high

Every CMSMS module must follow a standard folder structure. Required files include the main module class, install/uninstall methods, readme, language file, license, documentation, and an SVG icon. Missing files prevent proper installation, display, and distribution.

**Fix:** Create the missing files/directories according to the CMSMS module structure standard.

```
// Bad
modules/MyModule/
├── MyModule.module.php
└── lang/en_US.php
(missing: method.install.php, method.uninstall.php, readme.md, docs/, assets/)
```
```
// Good
modules/MyModule/
├── MyModule.module.php
├── method.install.php
├── method.uninstall.php
├── readme.md
├── action.defaultadmin.php
├── assets/ (banners, icons, screenshots)
├── docs/ (LICENSE, changelog.inc, help.inc)
├── lang/en_US.php
├── lib/
└── templates/
```

### CMSMS\_CMS\_023: PHP file missing CMS\_VERSION direct access guard

**Severity:** high

Every module PHP file (action files, method files, lib classes) must prevent direct browser access. Without the CMS\_VERSION guard, requesting the file directly can trigger PHP errors that reveal file paths, class structures, or partial logic execution. The main .module.php file is excluded because it extends CMSModule which handles this internally.

**Fix:** Add this line at the top of the file, after the opening <?php tag and any comments: if (!defined('CMS\_VERSION')) exit;

```php
// Bad
&lt;?php
// action.defaultadmin.php
$db = cmsms()-&gt;GetDb();
```
```php
// Good
&lt;?php
if (!defined('CMS_VERSION')) exit;
$db = cmsms()-&gt;GetDb();
```

### CMSMS\_CMS\_024: Class or function defined without namespace or module prefix

**Severity:** medium

CMSMS modules share a global PHP namespace. Defining a class or function without a namespace or module name prefix will cause fatal errors if another module uses the same name. Use PHP namespaces or prefix all names with the module name.

**Fix:** Add a namespace declaration (e.g. namespace MyModule;) or prefix the class/function name with the module name.

```php
// Bad
class Helper { ... }
function format_date($ts) { ... }
```
```php
// Good
namespace MyModule;
class Helper { ... }
// or
class MyModule_Helper { ... }
function mymodule_format_date($ts) { ... }
```

### CMSMS\_CMS\_025: PHP file missing license header comment

**Severity:** info

Every PHP file in a CMSMS module should include a license header comment near the top referencing the docs/LICENSE file. This ensures legal clarity for anyone reading or reusing the code. The standard header is: #-- See LICENSE for full license information. --#

**Fix:** Add the license header after the opening <?php tag:
<?php
#--------------------------------------------------
# See LICENSE for full license information.
#--------------------------------------------------

```php
// Bad
&lt;?php
if (!defined('CMS_VERSION')) exit;
$db = cmsms()-&gt;GetDb();
```
```php
// Good
&lt;?php
#--------------------------------------------------
# See LICENSE for full license information.
#--------------------------------------------------
if (!defined('CMS_VERSION')) exit;
$db = cmsms()-&gt;GetDb();
```

### CMSMS\_CMS\_026: Uninstall does not clean up resources created by install

**Severity:** high

A module that creates database tables, permissions, preferences, or events during install must clean them up on uninstall. Orphaned data wastes space, orphaned permissions clutter the admin UI, and orphaned tables can cause conflicts if the module is reinstalled or another module reuses the name.

**Fix:** For every CreatePermission() in install, add RemovePermission() in uninstall. For every CreateTableSQL(), add DropTableSQL(). For every SetPreference(), add RemovePreference(). For every CreateEvent(), add RemoveEvent().

```php
// Bad
// method.install.php
$this-&gt;CreatePermission('Manage MyModule');
// method.uninstall.php
// (empty - permissions left behind)
```
```php
// Good
// method.install.php
$this-&gt;CreatePermission('Manage MyModule');
// method.uninstall.php
$this-&gt;RemovePermission('Manage MyModule');
```

### CMSMS\_CMS\_027: Database table creation does not follow CMSMS conventions

**Severity:** high

CMSMS provides a database abstraction layer (ADODB Data Dictionary) for creating and modifying tables. Using raw CREATE TABLE or ALTER TABLE SQL bypasses this layer, breaking portability across database engines. Tables must use cms\_db\_prefix() for the table prefix and follow the module\_<modulename>\_ naming convention. InnoDB is required for foreign key support and crash recovery. Upgrades must use AddColumnSQL/AlterColumnSQL to safely modify existing tables.

**Fix:** Use $dict = NewDataDictionary($db); $sqlarray = $dict->CreateTableSQL(cms\_db\_prefix().'module\_mymod\_items', $flds, $taboptarray); $dict->ExecuteSQLArray($sqlarray); with $taboptarray = ['mysql' => 'ENGINE=InnoDB'];

```sql
// Bad
$db-&gt;Execute('CREATE TABLE module_mymod_items (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255))');
```
```php
// Good
$db = $this-&gt;GetDb();
$dict = NewDataDictionary($db);
$taboptarray = ['mysql' =&gt; 'ENGINE=InnoDB'];
$flds = 'id I KEY AUTO, name C(255) NOTNULL';
$sqlarray = $dict-&gt;CreateTableSQL(cms_db_prefix().'module_mymod_items', $flds, $taboptarray);
$dict-&gt;ExecuteSQLArray($sqlarray);
```

### CMSMS\_CMS\_028: Direct $\_GET access instead of $params in action file

**Severity:** low

CMSMS auto-cleans $\_GET and delivers action parameters via $params. Direct $\_GET access in action files bypasses module parameter scoping and may read parameters intended for other modules. $\_POST/$\_REQUEST access is acceptable since CMSMS 2.2.8 deprecated SetParameterType() in favor of direct input cleaning.

**Fix:** Use $params['key'] for action parameters. For direct input cleaning, use filter\_input(INPUT\_GET, 'key', FILTER\_SANITIZE\_...) instead of $\_GET['key'].

```
// Bad
$page = $_GET['page'];
```
```
// Good
$page = (int)($params['page'] ?? 1);
// Or for non-action context:
$page = filter_input(INPUT_GET, 'page', FILTER_VALIDATE_INT) ?: 1;
```

### CMSMS\_CMS\_030: Module missing permission constants

**Severity:** medium

Modern CMSMS modules define permission names as class constants (e.g. const MANAGE\_PERM = 'Manage MyModule') and reference them throughout the module. This prevents typos, enables IDE autocompletion, and makes permission names easy to change.

**Fix:** Add constants to the module class and reference them in CheckPermission(), CreatePermission(), and VisibleToAdminUser().
const MANAGE\_PERM = 'Manage MyModule';
const USE\_PERM = 'Use MyModule';

```php
// Bad
$this-&gt;CheckPermission('Manage MyModule');
$this-&gt;CreatePermission('Manage MyModule');
```
```php
// Good
const MANAGE_PERM = 'Manage MyModule';
$this-&gt;CheckPermission(self::MANAGE_PERM);
$this-&gt;CreatePermission(self::MANAGE_PERM);
```

### CMSMS\_CMS\_031: Deprecated $this->smarty module property

**Severity:** medium

The $this->smarty magic property on module objects was deprecated in CMSMS 2.1. Use cmsms()->GetSmarty() or the $smarty variable provided in action scope.

**Fix:** Replace $this->smarty with cmsms()->GetSmarty() or use the $smarty variable already available in action files.

```php
// Bad
$this-&gt;smarty-&gt;assign('data', $data);
```
```
// Good
$smarty = cmsms()-&gt;GetSmarty();
$smarty-&gt;assign('data', $data);
```

### CMSMS\_CMS\_032: Deprecated module parameter registration methods

**Severity:** medium

SetParameterType(), CreateParameter(), RestrictUnknownParameters(), and GetModuleParameters() were deprecated in CMSMS 2.2.8. Modules should now clean parameters directly using filter\_var() or type casting. In the future, $params in module actions will only consist of parameters passed on the module tag.

**Fix:** Remove SetParameterType/CreateParameter/RestrictUnknownParameters calls. Validate $params values directly with filter\_var(), (int) casting, or htmlspecialchars().

```php
// Bad
$this-&gt;SetParameterType('id', CLEAN_INT);
$this-&gt;CreateParameter('sortby', 'name', 'Sort field');
```
```
// Good
// In action file, validate directly:
$id = (int)($params['id'] ?? 0);
$sortby = in_array($params['sortby'] ?? '', ['name','date'], true) ? $params['sortby'] : 'name';
```

### CMSMS\_CMS\_033: Deprecated GetNotificationOutput() method

**Severity:** medium

GetNotificationOutput() was removed in CMSMS 2.2. The old notification system was replaced with the Admin Alerts classes (CmsAdminAlert).

**Fix:** Remove GetNotificationOutput(). Use the Admin Alerts API: create a class extending CmsAdminAlert and register it.

```
// Bad
function GetNotificationOutput() { return 'You have 3 pending items'; }
```
```
// Good
// Use Admin Alerts instead:
// Create a class extending CmsAdminAlert
// Register via alert_manager::load_alerts()
```

### CMSMS\_CMS\_034: Deprecated cms\_html\_entity\_decode() function

**Severity:** low

cms\_html\_entity\_decode() was deprecated in CMSMS 2.2.20 and is scheduled for removal. PHP's native html\_entity\_decode() now handles UTF-8 correctly.

**Fix:** Replace with html\_entity\_decode($string, ENT\_QUOTES, 'UTF-8').

```
// Bad
$text = cms_html_entity_decode($input);
```
```
// Good
$text = html_entity_decode($input, ENT_QUOTES, 'UTF-8');
```

### CMSMS\_CMS\_035: Deprecated strftime() function

**Severity:** medium

PHP's strftime() was deprecated in PHP 8.1 and removed in PHP 9.0. CMSMS 2.2.16+ provides locale\_ftime() as a drop-in replacement, and CMSMS 2.2.17+ provides CMSMS\strftime as a fallback. For Smarty templates, use the |localedate\_format modifier instead of |date\_format.

**Fix:** Replace strftime() with locale\_ftime() in PHP code. In templates, replace |date\_format with |localedate\_format.

```
// Bad
$date = strftime('%B %d, %Y', $timestamp);
```
```
// Good
$date = locale_ftime('%B %d, %Y', $timestamp);
```

### CMSMS\_CMS\_036: Dependency on removed core module MenuManager

**Severity:** high

MenuManager was removed from the CMSMS core package in version 2.2.22. It was replaced by Navigator. Templates using {MenuManager} will break on fresh installs.

**Fix:** Replace {MenuManager} with {Navigator} in templates. Update GetDependencies() to reference Navigator instead.

```smarty
// Bad
{MenuManager action='default' template='simple_navigation'}
```
```smarty
// Good
{Navigator action='default' template='simple_navigation'}
```
