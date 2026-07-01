## Communicating with Other Modules

CMSMS modules do not operate in isolation. A gallery module might use Captcha for form protection, a booking module might integrate with MAMS for authentication, or a custom module might read data from News. This chapter covers how to safely obtain a reference to another module and call its public methods.

### Getting a Module Instance

The primary way to communicate with another module is to get a reference to its object using `$this->GetModuleInstance()`:

```php
$captcha = $this->GetModuleInstance('Captcha');
```

This returns the module object if the module is installed, active, and loaded. If the module is not available, it returns `FALSE`. Always check the return value before calling methods on it.

An equivalent utility function is available via `\cms_utils::get_module()`:

```php
$captcha = \cms_utils::get_module('Captcha');
```

Both do exactly the same thing. Use `$this->GetModuleInstance()` when you are inside a module class or action file (where `$this` is available). Use `\cms_utils::get_module()` when you are in a static context, a library class, or anywhere outside the module scope.

> **Note:** There is also `\CmsApp::get_instance()->GetModuleInstance('ModuleName')`, which is the original method from CMSMS 1.x. It still works but is considered deprecated.

### Never Use require or include

Do not use `require`, `require_once`, `include`, or `include_once` to load files from another module:

```php
// BAD — never do this
require_once dirname(__DIR__) . '/OtherModule/lib/some_class.php';
```

This creates a brittle path dependency, bypasses CMSMS's module loading system, and breaks if the other module is not installed, gets renamed, or changes its internal file structure. It also circumvents version checks and dependency management entirely.

Always use `GetModuleInstance()` to interact with another module through its public API. If you need functionality from another module, call its methods. Do not include its files.

### Always Check Before Calling

Never assume a module is available. Even if you declare it as a dependency, defensive coding prevents fatal errors during edge cases like partial upgrades:

```php
$captcha = $this->GetModuleInstance('Captcha');
if (!is_object($captcha)) return;

$html = $captcha->getCaptcha();
```

`GetModuleInstance()` returns `FALSE` when the module is not installed or not active. If you skip the check, PHP will throw a fatal error when you try to call a method on a non-object.

### Hard Dependencies vs. Soft Dependencies

There are two strategies for inter-module communication, and the choice depends on whether your module can function without the other module.

#### Hard Dependency (required module)

If your module cannot function without another module, declare it in `GetDependencies()`:

```php
public function GetDependencies()
{
    return ['Captcha' => '1.0'];
}
```

This tells CMSMS:
- The other module must be installed before yours can be installed.
- If the other module is uninstalled, yours will be deactivated.
- The minimum version you require is enforced at install time.

With a hard dependency declared, CMSMS guarantees the module is loaded before yours. You still use `$this->GetModuleInstance()` to get the reference, but you have assurance it will be there.

Example: MicroTiny depends on FileManager because it cannot function without the file browser.

#### Soft Dependency (optional integration)

If your module works fine on its own but gains extra functionality when another module is present, do not declare a dependency. Instead, check at runtime:

```php
// Optional Search integration — works with or without Search module
$search = $this->GetModuleInstance('Search');
if (is_object($search)) {
    $search->AddWords($this->GetName(), $item_id, 'item', $text);
}
```

This pattern is common for:
- Search integration (indexing your content)
- Captcha (form protection)
- MAMS (checking if a frontend user is logged in)
- WYSIWYG editors (providing rich text fields)

The module works without the optional module. The feature simply is not available.

### What You Can Call

Once you have a module reference, you can call any of its **public methods**. Common patterns include:

```php
// Use Captcha for form verification
$captcha = $this->GetModuleInstance('Captcha');
if (is_object($captcha)) {
    $captcha_html = $captcha->getCaptcha();
    // ... render $captcha_html in your form template ...
    // On form submit, verify:
    $is_valid = $captcha->checkCaptcha($params['captcha_input']);
}

// Check frontend user login via MAMS
$mams = $this->GetModuleInstance('MAMS');
if (is_object($mams)) {
    $user_id = $mams->LoggedInId();
    if ($user_id) {
        // User is logged in
    }
}

// Add content to Search index
$search = $this->GetModuleInstance('Search');
if (is_object($search)) {
    $search->AddWords($this->GetName(), $record_id, 'item', $searchable_text);
}
```

### What You Must Not Do

Inter-module communication has strict boundaries:

- **Never** write directly to another module's database tables. Use its public API methods instead.
- **Never** modify another module's files, templates, or preferences directly.
- **Never** call methods that start with an underscore or are marked `@internal`. These are internal implementation details that may change without notice.
- **Never** rely on another module's internal class structure (its `lib/` classes) unless those classes are explicitly documented as part of its public API.

```php
// BAD — writing to News tables directly
$db->Execute('INSERT INTO ' . CMS_DB_PREFIX . 'module_news ...');

// GOOD — use the module's own methods
$news = $this->GetModuleInstance('News');
if (is_object($news)) {
    // Call whatever public API News exposes
}
```

### Version Checking

If you need a specific version of a module for a particular method, you can check it at runtime:

```php
$captcha = $this->GetModuleInstance('Captcha');
if (is_object($captcha) && version_compare($captcha->GetVersion(), '2.0', '>=')) {
    // Use a method only available in Captcha 2.0+
    $captcha->NewMethodInV2();
}
```

For hard dependencies, the version is enforced at install time via `GetDependencies()`. For soft dependencies, check at runtime if you need a specific version's features.

### Checking Module Availability Without Loading

If you only need to know whether a module exists (without getting a full reference), use:

```php
if (\cms_utils::module_available('Captcha')) {
    // Captcha is installed and active
}
```

This is lighter than `GetModuleInstance()` because it does not instantiate the module object. Use it when you need a boolean check (for example, to show or hide a UI option) but do not need to call any methods yet.

### The Capability System

Sometimes you do not need a specific module. Instead, you need any module that provides a certain type of functionality. CMSMS handles this with the capability system.

#### Declaring a Capability

A module advertises its capabilities by overriding `HasCapability()`:

```php
public function HasCapability($capability, $params = array())
{
    switch ($capability) {
        case 'wysiwyg':
            return true;
        case 'search':
            return true;
        default:
            return false;
    }
}
```

#### Finding Modules by Capability

To find all modules that provide a specific capability:

```php
$wysiwyg_modules = $this->GetModulesWithCapability('wysiwyg');
// Returns an array of module names, e.g. ['MicroTiny']
```

This is used internally by CMSMS to find the active WYSIWYG editor, search indexer, syntax highlighter, and similar pluggable components. You can define your own capability strings for your ecosystem of modules.

#### Common Built-in Capabilities

| Capability | Purpose | Example Modules |
|---|---|---|
| `wysiwyg` | Rich text editor | MicroTiny |
| `search` | Content indexing | Search |
| `AdminSearch` | Admin panel search provider | News, Content |
| `tasks` | Background task provider | Various |

### Exposing Your Own Public API

If you want other modules to communicate with yours, design a clear public API. Keep these methods in your main module class (the `CMSModule` subclass), document them, and keep them stable across versions:

```php
class Holidays extends CMSModule
{
    /**
     * Get all published holidays.
     * Public API — safe for other modules to call.
     *
     * @return array of HolidayItem objects
     */
    public function GetPublishedHolidays()
    {
        $query = new HolidayQuery(['status' => 'published']);
        $query->execute();
        return $query->GetMatches();
    }

    /**
     * Get a single holiday by ID.
     * Public API — safe for other modules to call.
     *
     * @param int $id
     * @return HolidayItem|null
     */
    public function GetHoliday(int $id)
    {
        return HolidayItem::load_by_id($id);
    }
}
```

Other modules can then use your API:

```php
$holidays = $this->GetModuleInstance('Holidays');
if (is_object($holidays)) {
    $items = $holidays->GetPublishedHolidays();
    foreach ($items as $item) {
        // Use the holiday data
    }
}
```

### Events vs. Direct Calls

CMSMS provides two ways for modules to communicate:

| Approach | Use When |
|---|---|
| `$this->GetModuleInstance()` + method call | You need to actively request data or trigger an action in a specific module |
| Events / Hooks | You want to notify any interested module that something happened, without knowing (or caring) who is listening |

Direct calls are for "I need something from module X." Events are for "something happened, and whoever cares can react." Use events when you want loose coupling. Use direct calls when you need a specific response from a specific module.

See the [Events chapter](/docs/events/events-overview) for the event/hook system.

### Summary

| Method | Returns | Purpose |
|---|---|---|
| `$this->GetModuleInstance('Name')` | Module object or `FALSE` | Get a reference to call methods on |
| `\cms_utils::get_module('Name')` | Module object or `null` | Same thing, for use outside module context |
| `\cms_utils::module_available('Name')` | `bool` | Check existence without loading |
| `$this->GetModulesWithCapability('cap')` | `array` of module names | Find all modules with a capability |
| `$this->GetDependencies()` | `array` | Declare hard dependencies |
| `HasCapability($cap)` | `bool` | Advertise your module's capabilities |
